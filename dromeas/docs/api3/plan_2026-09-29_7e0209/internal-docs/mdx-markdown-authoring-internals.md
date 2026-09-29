# MDX & Markdown Authoring Internals

## Overview

MDX and Markdown authoring in Docusaurus is powered by the `@docusaurus/mdx-loader` package. The package integrates MDX compilation into the webpack build pipeline and augments the standard MDX transformation with Docusaurus-specific remark plugins. These plugins resolve local asset references, rewrite Markdown nodes to JSX equivalents, extract table-of-contents data, and validate link/image integrity.

This document covers the modules, data flow, APIs, and extension points in the `packages/docusaurus-mdx-loader` package, based on the source files available in the repository at `main`.

---

## Architecture

### Pipeline overview

1. A webpack rule matches `.md` and `.mdx` files using the test expression `\.mdx?$/i`.
2. The rule invokes a webpack loader item created by `createMDXLoaderItem`.
3. The loader runs a unified processor built from Docusaurus remark plugins.
4. Each remark plugin visits the MDAST tree, mutates nodes in place, and produces an MDX output where local assets become `require` expressions and certain Markdown nodes become JSX elements.

### Package layout

The remark plugin modules live under:

```
packages/docusaurus-mdx-loader/src/remark/
├── admonitions/
├── contentTitle/
├── mermaid/
├── toc/
├── transformImage/
├── transformLinks/
└── utils/
```

Other core modules:

```
packages/docusaurus-mdx-loader/src/
├── createMDXLoader.ts
├── processor.ts           (referenced by createMDXLoader)
├── index.ts               (webpack loader entry point)
└── options.ts             (Options type)
```

---

## Webpack loader integration

File: `packages/docusaurus-mdx-loader/src/createMDXLoader.ts`

### `normalizeOptions`

```ts
function normalizeOptions(optionsInput: Options & CreateOptions): Options {
  if (process.env.NODE_ENV === 'test' || process.env.VITEST) {
    return optionsInput;
  }

  let options = optionsInput;

  if (!options.processors) {
    options = {...options, processors: createProcessors({options})};
  }

  if (options.useCrossCompilerCache && process.env.NODE_ENV === 'production') {
    options = {
      ...options,
      crossCompilerCache: new Map(),
    };
  }

  return options;
}
```

Behavior:

- In test environments (`NODE_ENV === 'test'` or `process.env.VITEST`), returns the input options unchanged and skips eager processor creation.
- Creates MDX processors eagerly when `options.processors` is not already provided. The comment states this avoids lazy creation, which disrupts Rsdoctor's ability to measure `mdx-loader` performance.
- When `useCrossCompilerCache` is true and `NODE_ENV === 'production'`, assigns a `Map` instance as `crossCompilerCache`. The cache permits compiling client and server MDX only once. The code comments note that multiple compilers only exist in production (`docusaurus build`), not dev (`docusaurus start`).

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

Returns a webpack `RuleSetUseItem` that points to the loader entry file `./index` and carries normalized options.

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

Creates a full webpack `RuleSetRule`:

- `test: /\.mdx?$/i` matches both `.md` and `.mdx` files case-insensitively.
- `include` is caller-controlled to scope the rule to specific directories.
- Uses a single loader item produced by `createMDXLoaderItem`.

Creating the rule is separate from the loader item, which lets callers keep the test/include contract explicit while reusing the same loader configuration.

---

## Shared remark utilities

File: `packages/docusaurus-mdx-loader/src/remark/utils/index.ts`

### In-place node transformation

```ts
export function transformNode<NewNode extends Node>(
  node: Node,
  newNode: NewNode,
): NewNode {
  Object.keys(node).forEach((key) => {
    delete node[key];
  });
  Object.keys(newNode).forEach((key) => {
    node[key] = newNode[key];
  });
  return node as NewNode;
}
```

`transformNode` mutates an existing unist node into a different node type. It deletes every property from the original node and copies every property from `newNode` into it. This is used by `transformImage` to convert Markdown `image` nodes into `mdxJsxTextElement` nodes, and by `transformLinks` to convert `link` nodes into `mdxJsxTextElement` anchors. The mutation preserves the node's original position in the tree while replacing its type and data.

### Asset require attribute values

```ts
export function assetRequireAttributeValue(
  requireString: string,
  hash: string,
): MdxJsxAttributeValueExpression {
  return {
    type: 'mdxJsxAttributeValueExpression',
    value: `require("${requireString}").default${hash && ` + '${hash}'`}`,
    data: {
      estree: { /* ExpressionStatement with BinaryExpression */ },
    },
  };
}
```

Builds an MDX JSX attribute value expression representing:

```js
require("<requireString>").default + '<hash>'
```

When `hash` is empty, the `+ '<hash>'` suffix is omitted through short-circuit evaluation. The returned object includes both a `value` string form and an `estree` program describing the expression:

- `ExpressionStatement`
- `BinaryExpression` with operator `+`
- Left side is a `MemberExpression` accessing `.default` on a `require()` call
- Right side is a string `Literal` containing the hash

### Error position formatting

```ts
export function formatNodePositionExtraMessage(node: Node): string {
  const position = formatNodePosition(node);
  return `${position ? ` (${position})` : ''}`;
}
```

Returns a string such as ` (12:5)` for the node's start line and column when position data is available. The leading space lets the message be appended to existing log text without producing awkward concatenation. The private `formatNodePosition` uses `@docusaurus/logger` interpolation to format the line and column as `number=12:number=5`.

---

## Image transformation plugin

File: `packages/docusaurus-mdx-loader/src/remark/transformImage/index.ts`

### Plugin options and context

```ts
export type PluginOptions = {
  staticDirs: string[];
  siteDir: string;
  onBrokenMarkdownImages: MarkdownConfig['hooks']['onBrokenMarkdownImages'];
};

type Context = {
  staticDirs: PluginOptions['staticDirs'];
  siteDir: PluginOptions['siteDir'];
  onBrokenMarkdownImages: OnBrokenMarkdownImagesFunction;
  filePath: string;
  inlineMarkdownImageFileLoader: string;
};
```

The plugin accepts:

- `staticDirs` — absolute paths where static assets may live.
- `siteDir` — the site root directory, used to resolve `@site/` URLs.
- `onBrokenMarkdownImages` — either a string policy or a user-supplied callback.

At transform time the context adds:

- `filePath` — the absolute path of the Markdown source file from `vfile.path`.
- `inlineMarkdownImageFileLoader` — a loader string obtained from `getFileLoaderUtils(vfile.data.compilerName === 'server')`. When the compiler is the server compiler, the returned loader is the server-appropriate inline image loader.
- A normalized `onBrokenMarkdownImages` function.

### Broken image hook normalization

```ts
function asFunction(
  onBrokenMarkdownImages: PluginOptions['onBrokenMarkdownImages'],
): OnBrokenMarkdownImagesFunction
```

When `onBrokenMarkdownImages` is the string `'throw'` or `'warn'`, the plugin returns a function that reports a diagnostic through `logger.report`. The throw variant appends guidance:

```
To ignore this error, use the siteConfig.markdown.hooks.onBrokenMarkdownImages option, or apply the pathname:// protocol to the broken image URLs.
```

When it is already a function, the wrapper passes through parameters but normalizes `sourceFilePath` with `toMessageRelativeFilePath`.

### Converting images to JSX requires

`toImageRequireNode` receives the image target, an absolute local image path, and the plugin context. It:

1. Casts the Markdown `image` node to an `MdxJsxTextElement` so it can be mutated.
2. Computes the relative posix path from the source file's directory to the image file, prefixed with `./`.
3. Parses the original node URL to extract `hash` and `search` query parameters.
4. Builds `requireString` as:

```ts
`${context.inlineMarkdownImageFileLoader}${escapePath(relativeImagePath) + search}`
```

5. Adds `alt`, `src`, and `title` MDX JSX attributes. The `src` attribute uses `assetRequireAttributeValue(requireString, hash)`.
6. Attempts to read image width and height via `imageSizeFromFile(imagePath)` and adds `width`/`height` attributes when available. If reading fails, it logs a warning unless running under Yarn PnP (`process.versions.pnp`), where the error is intentionally suppressed to work around a known PnP issue.
7. Calls `transformNode` to replace the original image node with:

```ts
{
  type: 'mdxJsxTextElement',
  name: 'img',
  attributes,
  children: [],
}
```

### Local image path resolution

`getLocalImageAbsolutePath` resolves three forms of image URL:

| Form | Resolution strategy |
|---|---|
| `@site/<path>` | `path.join(siteDir, path)` |
| Absolute path | Try `path.join(dir, originalImagePath)` against each `staticDirs` entry, using `findAsyncSequential` |
| Relative path | `path.join(path.dirname(filePath), originalImagePath)` |

Each candidate is verified with `fs.pathExists`. Returns `null` when no candidate exists.

### Node processing flow

`processImageNode`:

1. If `node.url` is empty or falsy, calls `onBrokenMarkdownImages` and assigns its return value (or the original URL if the hook returns `null` or `undefined`).
2. Parses the URL with `parseLocalURLPath`. If not a local URL:
   - If the protocol is `pathname:`, strips `pathname://` and returns. This protocol is an escape hatch that prevents webpack require conversion.
   - Otherwise returns without changing the node.
3. Decodes the pathname with `decodeURIComponent`. The comment explains that Node's `Url.pathname` is always encoded while filesystem paths are not; see discussion `#10720`.
4. Calls `getLocalImageAbsolutePath`.
5. On failure, invokes `onBrokenMarkdownImages` and uses its return value.
6. On success, calls `toImageRequireNode` to mutate the node.

### Plugin export

The default export is a unified `Plugin<PluginOptions[], Root>` returning a `Transformer<Root>`. In `transform`:

- Creates the context using current `vfile`.
- Visits all `image` nodes with `unist-util-visit`.
- Pushes each `processImageNode` promise into an array.
- Awaits `Promise.all(promises)`, so all image resolutions complete before the AST continues compilation.

---

## Link transformation plugin

File: `packages/docusaurus-mdx-loader/src/remark/transformLinks/index.ts`

### Plugin options and context

The structure mirrors transformImage. Options include `staticDirs`, `siteDir`, and `onBrokenMarkdownLinks`. The context adds `filePath` and `inlineMarkdownLinkFileLoader`, obtained from `getFileLoaderUtils(vfile.data.compilerName === 'server')`.

### Broken link hook normalization

`asFunction` performs the same string-to-function conversion as in transformImage, with the throw variant appending:

```
To ignore this error, use the siteConfig.markdown.hooks.onBrokenMarkdownLinks option, or apply the pathname:// protocol to the broken link URLs.
```

### Converting links to asset requires

`toAssetRequireNode` converts a Markdown `link` node to an `<a>` JSX element. Key details:

1. Computes `relativeAssetPath` as:

```ts
`./${posixPath(path.relative(path.dirname(context.filePath), assetPath))}`
```

2. Extracts `hash` and `search` from the original URL.
3. Builds the require string with a special JSON workaround:

```ts
const requireString = `${
  path.extname(relativeAssetPath) === '.json'
    ? `${relativeAssetPath.replace('.json', '.raw')}!=`
    : ''
}${context.inlineMarkdownLinkFileLoader}${
  escapePath(relativeAssetPath) + search
}`;
```

When the linked file has a `.json` extension, the path is rewritten from `.json` to `.raw` followed by `!=`. The comment identifies this as a hack to stop webpack from applying its built-in JSON loader.

4. Adds these attributes:
   - `target` set to `_blank`
   - `data-noBrokenLinkCheck` as an expression attribute evaluating to `true` — this prevents the broken link checker from reprocessing an asset link that has already been validated by this plugin
   - `href` via `assetRequireAttributeValue(requireString, hash)`
   - `title` when present, HTML-escaped with `escapeHtml`
5. Preserves the link node's children when calling `transformNode` to build the `<a>` element.

### Link target file resolution

`isExistingFile` returns true only when `fs.stat(filePath).isFile()` succeeds; any error returns false.

`getLocalFileAbsolutePath` mirrors image resolution:

| URL form | Strategy |
|---|---|
| `@site/<path>` | Join `siteDir` with the path, verify with `isExistingFile` |
| Absolute path | Search `staticDirs` with `findAsyncSequential` and `isExistingFile` |
| Relative path | Join `dirname(filePath)` with the path, verify with `isExistingFile` |

Returns `null` if no existing file is found.

### Asset link eligibility

`processLinkNode` only converts a link to an asset require when:

- The URL is a local URL parsed by `parseLocalURLPath`.
- Either the pathname starts with `@site/`, or the pathname has a file extension that is not `md`, `mdx`, or `html`.

When the `@site/` alias is used and the file cannot be resolved, `onBrokenMarkdownLinks` is invoked. When the URL only has an asset-like extension without the alias, no broken link report is emitted because a route path segment can legitimately look like a filename. The code comments this decision to avoid risky fail-fast behavior by default.

### Plugin export

The transformer visits all `link` nodes and processes them in parallel through `Promise.all`. Links to local Markdown pages and regular HTTP URLs are left untouched by this plugin.

---

## Table-of-contents utilities

File: `packages/docusaurus-mdx-loader/src/remark/toc/utils.ts`

These utilities operate on the MDX ESTree program to add named imports and exports that carry the TOC data.

### Import declarations

```ts
export function getImportDeclarations(program: Program): ImportDeclaration[]

export function isMarkdownImport(node: Node): node is ImportDeclaration

export function findDefaultImportName(
  importDeclaration: ImportDeclaration,
): string | undefined

export function findNamedImportSpecifier(
  importDeclaration: ImportDeclaration,
  localName: string,
): ImportSpecifier | undefined
```

- `getImportDeclarations` filters the ESTree program body to `ImportDeclaration` nodes.
- `isMarkdownImport` returns true when the import source is a string that matches `/\.mdx?$/`.
- `findDefaultImportName` retrieves the local name from the default import specifier.
- `findNamedImportSpecifier` searches for an `ImportSpecifier` whose local name matches `localName`.

### Adding a TOC slice import

```ts
export function addTocSliceImportIfNeeded({
  importDeclaration,
  tocExportName,
  tocSliceImportName,
}: {
  importDeclaration: ImportDeclaration;
  tocExportName: string;
  tocSliceImportName: string;
}): void
```

Adds a named import specifier if it does not already exist. The imported identifier uses `tocExportName`, while the local identifier uses `tocSliceImportName`. This enables a source Markdown file to consume the exported TOC from an imported Markdown partial.

### Detecting a named export

```ts
export function isNamedExport(
  node: Node,
  exportName: string,
): node is MdxjsEsm
```

Returns true when the node is an `mdxjsEsm` node whose ESTree program has exactly one `ExportNamedDeclaration`, containing a `VariableDeclaration` whose first declared identifier name matches `exportName`.

### Creating the TOC export node

```ts
export function createTOCExportNodeAST({
  tocExportName,
  tocItems,
}: {
  tocExportName: string;
  tocItems: TOCItems;
}): MdxjsEsm
```

Builds an `mdxjsEsm` node representing:

```js
export const <tocExportName> = [
  ...<slice.importName>,
  {value: '<html>', id: '<id>', level: <depth>},
];
```

Implementation details:

- A `slice` TOC item becomes a `SpreadElement` whose argument is an identifier for the slice's import name.
- A `heading` TOC item becomes an object with:
  - `value` from `toHeadingHTMLValue(heading)`
  - `id` from `heading.data.id`
  - `level` from `heading.depth`
- The resulting `mdxjsEsm` node sets `value: ''` with a comment referencing PR `#9684`.
- `exportName` is declared with `const`.

### Heading HTML serialization

`toHeadingHTMLValue` converts phrase content into an HTML string:

| Node type | Output |
|---|---|
| `text` | HTML-escaped text |
| `inlineCode` | `<code>` with escaped value |
| `emphasis` | `<em>` with child content |
| `strong` | `<strong>` with child content |
| `delete` | `<del>` with child content |
| `link` | child content only, no anchor tag |
| `heading` | child content concatenated |
| `mdxJsxTextElement` | serialized via `mdxJsxTextElementToHtml` |
| default | `toString(node)` |

`mdxJsxTextElementToHtml` serializes an MDX JSX element to an HTML string. It specifically:

- Returns an empty string for `img` elements, referencing issue `#11003`.
- Only emits the `className` attribute, falling back to the `class` attribute when `className` is absent.
- Escapes the class value with `escapeHtml`.
- Recursively serializes children. The source acknowledges this is a workaround rather than a complete JSX-to-HTML implementation, limited to tag name, class name, and content.

---

## Content title and mermaid plugins

Two remark plugin modules are present in the repository under `packages/docusaurus-mdx-loader/src/remark/` but their full source was not available in the prefetched files:

| Module | File path | Size |
|---|---|---|
| Content title | `contentTitle/index.ts` | 2385 bytes |
| Mermaid | `mermaid/index.ts` | 1021 bytes |

Both modules also contain `__tests__` directories. These are remark plugin modules in the same pattern as `transformImage` and `transformLinks`, meaning they are unified plugins that operate on the MDAST tree. The full implementation details are not documented here because the source content for these files was not provided in the available context.

---

## Admonitions plugin

Present at `packages/docusaurus-mdx-loader/src/remark/admonitions/index.ts` (4166 bytes), with a `LICENSE` file and `README.md`. The presence of a separate LICENSE in the admonitions directory indicates that the admonitions syntax transformation incorporates third-party implementation logic. Full source content was not included in the prefetched files, so the precise internal behavior is not documented here.

---

## Data flow

The complete authoring-to-compiled-output flow is:

1. A source `.md` or `.mdx` file is read from disk.
2. The webpack rule created by `createMDXLoaderRule` matches the file with `/\.mdx?$/i`.
3. The loader item created by `createMDXLoaderItem` invokes `packages/docusaurus-mdx-loader/src/index`.
4. The loader creates MDX processors through `createProcessors` from `processor.ts`. In production with `useCrossCompilerCache`, a `Map` shared cache lets client and server compilation reuse the same MDX result.
5. The unified processor runs the remark plugins:
   - Admonitions transformation (module present, implementation not prefetched)
   - Content title extraction (module present, implementation not prefetched)
   - Mermaid diagram handling (module present, implementation not prefetched)
   - Table-of-contents utilities, which inject `export const toc`-style ESTree nodes and markdown partial import handling
   - Image transformation, which converts local references to webpack `require` calls
   - Link transformation, which converts local asset links to webpack `require` calls
6. Each plugin mutates the MDAST tree in place. `transformImage` and `transformLinks` run asynchronous file system checks and image dimension reads, awaiting `Promise.all` on all node visitors before continuing.
7. The transformed AST is compiled by MDX to JavaScript, where `require` calls are later resolved by webpack's asset loading pipeline.

---

## Extension points

### `onBrokenMarkdownImages` and `onBrokenMarkdownLinks`

Both `transformImage` and `transformLinks` accept hooks for broken resource handling. The hook types are:

```ts
MarkdownConfig['hooks']['onBrokenMarkdownImages']
MarkdownConfig['hooks']['onBrokenMarkdownLinks']
```

These accept either:

- A string policy: `'throw'` to fail the build, or `'warn'` to report without failing.
- A user-supplied function. The plugin wraps the function and normalizes `sourceFilePath` with `toMessageRelativeFilePath` before passing it to the caller's hook.

The hook receives `url`, `sourceFilePath`, and `node`. The return value, when non-null, becomes the replacement URL for the broken node.

### `pathname://` protocol escape hatch

Both link and image plugins honor `pathname://`. For images, the protocol is stripped and the URL remains a plain HTML reference that bypasses webpack `require` conversion. For links, the protocol is left alone and the node is skipped entirely so that the Docusaurus `<Link>` component handles it.

### `@site/` alias

Both plugins accept `@site/<path>` URLs, which are resolved relative to `siteDir`. This is the only reliable way for the link plugin to distinguish an intended local asset from a route segment that merely looks like a filename.

### `staticDirs`

Image and link resolution accepts one or more static directories. Absolute paths in Markdown URLs are resolved by searching each static dir entry sequentially with `findAsyncSequential`.

### Webpack loader rule construction

External packages can use `createMDXLoaderItem` or `createMDXLoaderRule` to integrate Docusaurus MDX processing into their own webpack configurations. `createMDXLoaderRule` is parameterized by `include` so callers control exactly which directories receive the Docusaurus MDX transformation.

### TOC integration for markdown partials

The TOC utilities expose named functions for integrating with imported markdown partials. Imported partials can export their own `toc` slice. The utilities add named imports such as `{toc as __tocPartial}` and generate an `export const toc` array that spreads those slice imports alongside the page's own headings. This enables automatic TOC aggregation across MDX partial boundaries.

---

## Error and diagnostic references

The transformation plugins report broken references using:

- `toMessageRelativeFilePath(sourceFilePath)` — converts absolute paths into project-relative paths for diagnostics.
- `formatNodePositionExtraMessage(node)` — appends `(line:column)` when position data exists.
- `logger.report(reportingSeverity)` — interpolated logging API that formats the plugin name and source path consistently.