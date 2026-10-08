# MDX & Markdown Authoring Internals

This document covers the internal modules, services, data flow, APIs, and extension points behind MDX and Markdown authoring in `packages/docusaurus-mdx-loader`. The source files referenced here are part of the `Mohita111/docusaurus` repository.

## Scope

The authoring pipeline lives in `packages/docusaurus-mdx-loader/src`. It covers:

- Webpack integration through `createMDXLoader.ts`
- Remark plugins that transform the MDAST before MDX compilation
- Shared AST utilities
- Broken link and broken image reporting hooks
- TOC import and export AST utilities

## Webpack integration

File: `packages/docusaurus-mdx-loader/src/createMDXLoader.ts`

This module creates the Webpack loader rule and loader item for `.md` and `.mdx` files.

### `createMDXLoaderItem`

```ts
export function createMDXLoaderItem(
  options: Options & CreateOptions,
): RuleSetUseItem {
  return {
    loader: require.resolve('./index'),
    options: normalizeOptions(options),
  };
}
```

The loader resolves to `./index`, and options are normalized before passing them to the loader.

### `createMDXLoaderRule`

```ts
export function createMDXLoaderRule({
  include,
  options,
}: {
  include: RuleSetRule['include'];
  options: Options & CreateOptions;
}): RuleSetRule {
  return {
    test: /\.mdx?$/i,
    include,
    use: [createMDXLoaderItem(options)],
  };
}
```

The regex `/\.mdx?$/i` matches both `.md` and `.mdx` files.

### `normalizeOptions`

`normalizeOptions` has three important behaviors:

1. It skips eager processor creation in tests when `process.env.NODE_ENV === 'test'` or `process.env.VITEST` is set.
2. It creates processors eagerly through `createProcessors({options})` when `options.processors` is not already provided.
3. It adds a shared `crossCompilerCache` when `useCrossCompilerCache` is true and `NODE_ENV === 'production'`.

```ts
if (options.useCrossCompilerCache && process.env.NODE_ENV === 'production') {
  options = {
    ...options,
    crossCompilerCache: new Map(),
  };
}
```

The cross-compiler cache is intended to compile client and server MDX only once during production builds.

## Shared AST utilities

File: `packages/docusaurus-mdx-loader/src/remark/utils/index.ts`

### `transformNode`

```ts
export function transformNode<NewNode extends Node>(
  node: Node,
  newNode: NewNode,
): NewNode
```

This utility mutates an existing unist node in place:

1. Deletes every key on the original node.
2. Copies every key from `newNode` onto the original node.
3. Returns the original node typed as `NewNode`.

This is used by link and image transforms to replace Markdown nodes with `mdxJsxTextElement` nodes.

### `assetRequireAttributeValue`

```ts
export function assetRequireAttributeValue(
  requireString: string,
  hash: string,
): MdxJsxAttributeValueExpression
```

This function builds an MDX JSX attribute value expression of the form:

```ts
require("${requireString}").default${hash && ` + '${hash}'`}
```

It also returns the accompanying ESTree AST, so the expression is ready to be embedded into an MDX AST.

### Position formatting

```ts
function formatNodePosition(node: Node): string | undefined
export function formatNodePositionExtraMessage(node: Node): string
```

`formatNodePositionExtraMessage` returns a string such as ` (line:column)` when positional info is available. The leading space is intentional so it can be appended directly to existing messages.

## Image transformation

File: `packages/docusaurus-mdx-loader/src/remark/transformImage/index.ts`

The image transform replaces Markdown `image` nodes with MDX JSX `<img>` elements that use Webpack `require()` calls.

### Plugin options

```ts
export type PluginOptions = {
  staticDirs: string[];
  siteDir: string;
  onBrokenMarkdownImages: MarkdownConfig['hooks']['onBrokenMarkdownImages'];
};
```

The plugin is a unified `Plugin<PluginOptions[], Root>`.

### Image processing flow

1. Visit every `image` node in the MDAST.
2. For each node, call `processImageNode`.
3. If `node.url` is empty, call `onBrokenMarkdownImages` and assign the returned value.
4. Parse the URL with `parseLocalURLPath`.
5. If the URL has the `pathname:` protocol, remove `pathname://` and return; this is the escape hatch for disabling image transformation.
6. Decode the local path with `decodeURIComponent`, because `URL.pathname` is encoded but file system paths are not.
7. Resolve the local image path with `getLocalImageAbsolutePath`.
8. If the path exists, call `toImageRequireNode`. Otherwise, call `onBrokenMarkdownImages`.

### `getLocalImageAbsolutePath`

The path is resolved in this order:

- `@site/` alias: the prefix is replaced with `siteDir`.
- Absolute path: searched inside every `staticDirs` entry using `findAsyncSequential`.
- Relative path: resolved against `path.dirname(filePath)`.

Each candidate must pass `fs.pathExists`.

### `toImageRequireNode`

The target `image` node is mutated into an `MdxJsxTextElement` named `img`. Attributes are built in this order:

- `alt`, escaped with `escapeHtml`
- `src`, created with `assetRequireAttributeValue`
- `title`, escaped with `escapeHtml`
- `width` and `height`, populated from `imageSizeFromFile`

The `src` require string is:

```ts
const requireString = `${context.inlineMarkdownImageFileLoader}${
  escapePath(relativeImagePath) + search
}`;
```

The relative path is prefixed with `./`.

### File loader selection

The plugin selects the appropriate Webpack loader with:

```ts
const fileLoaderUtils = getFileLoaderUtils(
  vfile.data.compilerName === 'server',
);
```

The resulting `inlineMarkdownImageFileLoader` is stored in the plugin context.

## Link transformation

File: `packages/docusaurus-mdx-loader/src/remark/transformLinks/index.ts`

The link transform replaces Markdown `link` nodes that target local asset files with MDX JSX `<a>` elements.

### Plugin options

```ts
export type PluginOptions = {
  staticDirs: string[];
  siteDir: string;
  onBrokenMarkdownLinks: MarkdownConfig['hooks']['onBrokenMarkdownLinks'];
};
```

### Link processing flow

1. Visit every `link` node.
2. If `node.url` is empty, call `onBrokenMarkdownLinks`.
3. Parse the URL with `parseLocalURLPath`.
4. If `parseLocalURLPath` returns null, return; `pathname://` is not processed here because it is used by the `<Link>` component.
5. Determine whether the link has the `@site/` alias or an asset-like extension.
6. If it has neither, return.
7. Decode the path with `decodeURIComponent`.
8. Resolve the local file path with `getLocalFileAbsolutePath`.
9. If the file exists, call `toAssetRequireNode`. Otherwise, if the link used `@site/`, call `onBrokenMarkdownLinks`.

The asset-like extension check is:

```ts
const hasAssetLikeExtension =
  path.extname(localUrlPath.pathname) &&
  !localUrlPath.pathname.match(/\.(?:mdx?|html)(?:#|$)/);
```

This avoids treating Markdown and HTML routes as assets.

### `toAssetRequireNode`

The link node is mutated into an `MdxJsxTextElement` named `a` with these attributes:

- `target` set to `_blank`
- `data-noBrokenLinkCheck` set to the expression `true`, preventing the broken link checker from processing this generated node
- `href` set with `assetRequireAttributeValue`
- `title`, if present, escaped with `escapeHtml`

For `.json` assets, `toAssetRequireNode` adds an inline Webpack loader hack:

```ts
const requireString = `${
  path.extname(relativeAssetPath) === '.json'
    ? `${relativeAssetPath.replace('.json', '.raw')}!=`
    : ''
}${context.inlineMarkdownLinkFileLoader}${
  escapePath(relativeAssetPath) + search
}`;
```

This uses the `!=` Webpack loader separator to avoid Webpack's built-in JSON loader.

### File loader selection

The plugin uses the same server-aware selection pattern:

```ts
const fileLoaderUtils = getFileLoaderUtils(
  vfile.data.compilerName === 'server',
);
```

The resulting `inlineMarkdownLinkFileLoader` is stored in the plugin context.

## TOC utilities

File: `packages/docusaurus-mdx-loader/src/remark/toc/utils.ts`

These utilities operate on ESTree and MDX AST nodes to generate and merge table-of-contents exports.

### Import detection and manipulation

- `getImportDeclarations(program: Program): ImportDeclaration[]`
- `isMarkdownImport(node: Node): node is ImportDeclaration`
- `findDefaultImportName(importDeclaration: ImportDeclaration): string | undefined`
- `findNamedImportSpecifier(importDeclaration: ImportDeclaration, localName: string): ImportSpecifier | undefined`
- `addTocSliceImportIfNeeded(...)`

`addTocSliceImportIfNeeded` adds a named import such as:

```ts
import Partial, {toc as __tocPartial} from "partial"
```

when the import specifier does not already exist.

### Export detection

`isNamedExport(node, exportName)` checks whether an MDX ESM node is a named export with a variable declaration whose identifier matches `exportName`.

### Export AST creation

`createTOCExportNodeAST({tocExportName, tocItems})` creates an `MdxjsEsm` node exporting a const array:

```ts
export const tocExportName = [...]
```

Each TOC item is represented by either:

- A `SpreadElement` referencing a slice import name
- An object with `value`, `id`, and `level` created by `valueToEstree`

The generated node sets `value: ''` on the `mdxjsEsm` node.

### Heading HTML serialization

`toHeadingHTMLValue` serializes supported heading content types:

- `mdxJsxTextElement`: serialized to HTML, except `<img>` returns an empty string
- `text`: HTML-escaped
- `inlineCode`: wrapped in `<code>`
- `emphasis`: wrapped in `<em>`
- `strong`: wrapped in `<strong>`
- `delete`: wrapped in `<del>`
- `link`: children only
- Any other node: `toString(node)`

The MDX JSX serializer only supports tag name, `className`/`class`, and content for now.

## Additional remark plugin directories

The following plugin directories exist under `packages/docusaurus-mdx-loader/src/remark`, but their implementation files were not included in the current source set.

### `admonitions`

Path: `packages/docusaurus-mdx-loader/src/remark/admonitions`

The directory contains:

- `index.ts`
- `__tests__`
- `LICENSE`
- `README.md`

The implementation of `index.ts` is not documented here because its source content was not provided.

### `contentTitle`

Path: `packages/docusaurus-mdx-loader/src/remark/contentTitle`

The directory contains:

- `index.ts`
- `__tests__`

The implementation of `index.ts` is not documented here because its source content was not provided.

### `mermaid`

Path: `packages/docusaurus-mdx-loader/src/remark/mermaid`

The directory contains:

- `index.ts`
- `__tests__`

The implementation of `index.ts` is not documented here because its source content was not provided.

## Hook behavior

Both image and link transforms accept `onBrokenMarkdownImages` and `onBrokenMarkdownLinks` hooks. The hooks can be either strings or functions.

When the hook is a string, `asFunction` converts it into a callback that uses `logger.report`. For `'throw'`, the callback includes additional help text referencing `siteConfig.markdown.hooks.onBrokenMarkdownImages` or `siteConfig.markdown.hooks.onBrokenMarkdownLinks`.

For `pathname://`, the image plugin explicitly removes the prefix and skips transformation. The link plugin leaves `pathname://` URLs untouched because they are handled by the `<Link>` component.

## Data flow

1. A `.md` or `.mdx` file is matched by the Webpack rule from `createMDXLoaderRule`.
2. Webpack invokes the loader resolved from `./index`.
3. The loader receives normalized options from `normalizeOptions`.
4. During non-test builds, processors are eagerly created by `createProcessors`.
5. Processor plugins traverse the MDAST:
   - Image nodes become MDX JSX `img` elements with `require()` calls.
   - Link nodes that target local asset files become MDX JSX `a` elements with `require()` calls.
   - TOC utilities modify ESM imports and exports.
   - Additional plugins for admonitions, content titles, and Mermaid are present in the remark directory.
6. Mutated MDX is passed to the MDX compiler for client and server compilations.

The exact plugin ordering is configured in `./processor`, which is not included in this source set.

## Extension points

The main extension points are:

- `staticDirs` and `siteDir` options control how local files are resolved.
- `onBrokenMarkdownImages` and `onBrokenMarkdownLinks` allow custom reporting or recovery behavior.
- `@site/` aliases point to files under `siteDir`.
- `pathname://` prevents image transformation for the image plugin.
- Unified `Plugin` options can be extended by adding or wrapping transform functions.
- The Webpack loader item can be customized through `createMDXLoaderItem` options.

## Known limitations

This documentation is based on the pre-fetched source files listed in the task request. The following files are referenced but not included:

- `packages/docusaurus-mdx-loader/src/index.ts`
- `packages/docusaurus-mdx-loader/src/processor.ts`
- `packages/docusaurus-mdx-loader/src/options.ts`
- `packages/docusaurus-mdx-loader/src/remark/admonitions/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/contentTitle/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/mermaid/index.ts`

Therefore, the exact processor pipeline order and the full plugin behavior of admonitions, content title, and Mermaid are not documented here.