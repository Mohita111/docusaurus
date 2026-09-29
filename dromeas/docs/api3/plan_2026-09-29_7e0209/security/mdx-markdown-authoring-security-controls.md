# MDX and Markdown authoring security controls

This document covers the security-relevant behavior of the Markdown and MDX authoring pipeline in the Docusaurus repository. It identifies authentication, authorization, data handling, secrets, and threat areas based on the source files under `packages/docusaurus-mdx-loader/src/`.

## Overview

MDX and Markdown content is processed at **build time** by the Docusaurus MDX loader. The loader transforms Markdown nodes into JSX and ESM nodes that Webpack can bundle. Security controls focus on:

- Path resolution for images and links
- HTML and JavaScript escaping during AST transformations
- Enforcement of broken link and image policies
- Preventing code injection through generated `require()` calls

The MDX loader does not include a runtime authentication or authorization subsystem. The security boundary is the source repository; content is assumed to come from trusted site authors unless otherwise specified.

## Scope

The analysis covers these source files:

- `packages/docusaurus-mdx-loader/src/remark/transformImage/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/transformLinks/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/toc/utils.ts`
- `packages/docusaurus-mdx-loader/src/remark/utils/index.ts`
- `packages/docusaurus-mdx-loader/src/createMDXLoader.ts`

The pre-fetched content for `remark/contentTitle/index.ts`, `remark/mermaid/index.ts`, and `remark/admonitions/index.ts` includes only directory metadata, not full source. Their security analysis is incomplete in this document.

## Authentication and authorization

No authentication or authorization mechanisms exist in the provided MDX authoring source code. The loader does not check user identity, roles, or permissions. It operates on local file contents passed through the unified/remark pipeline.

### Code evidence

- `transformImage/index.ts` processes all `'image'` nodes found by `unist-util-visit`.
- `transformLinks/index.ts` processes all `'link'` nodes.
- `createMDXLoader.ts` creates Webpack loaders based on a file extension test (`/\.mdx?$/i`) and no authorization checks.

**Security implication:** Authorization is the responsibility of the source control and build environment, not the MDX loader itself.

## Data handling

### File path resolution

Both `transformImage/index.ts` and `transformLinks/index.ts` resolve local asset paths using a three-case strategy.

#### Images — `getLocalImageAbsolutePath`

Location: `packages/docusaurus-mdx-loader/src/remark/transformImage/index.ts`

```ts
if (originalImagePath.startsWith('@site/')) {
  const imageFilePath = path.join(siteDir, originalImagePath.replace('@site/', ''));
  ...
} else if (path.isAbsolute(originalImagePath)) {
  const possiblePaths = staticDirs.map((dir) => path.join(dir, originalImagePath));
  ...
} else {
  const imageFilePath = path.join(path.dirname(filePath), originalImagePath);
  ...
}
```

#### Links — `getLocalFileAbsolutePath`

Location: `packages/docusaurus-mdx-loader/src/remark/transformLinks/index.ts`

```ts
if (assetPath.startsWith('@site/')) {
  const assetFilePath = path.join(siteDir, assetPath.replace('@site/', ''));
  ...
} else if (path.isAbsolute(assetPath)) {
  const assetFilePath = await findAsyncSequential(
    staticDirs.map((dir) => path.join(dir, assetPath)),
    isExistingFile,
  );
  ...
} else {
  const assetFilePath = path.join(path.dirname(filePath), assetPath);
  ...
}
```

Both functions use `path.join` to combine the base directory with the user-provided path. There is no validation that the resolved path remains inside `siteDir`, `staticDirs`, or the source file directory.

**Threat:** A Markdown image or link URL containing `../` can escape the intended directory. For example, `../../etc/passwd` could resolve outside the site folder if the build process has filesystem access and the file exists.

### URL decoding before filesystem access

Both `processImageNode` and `processLinkNode` call `decodeURIComponent()` on the URL pathname before resolving the file path.

Image example:

```ts
const decodedPathname = decodeURIComponent(localUrlPath.pathname);
const localImagePath = await getLocalImageAbsolutePath(decodedPathname, context);
```

Link example:

```ts
const localFilePath = await getLocalFileAbsolutePath(
  decodeURIComponent(localUrlPath.pathname),
  context,
);
```

This decodes percent-encoded characters such as `%2e%2e%2f` into `../`. No path normalization or containment check follows the decoding step.

**Threat:** Crafted Markdown content can use percent-encoded traversal sequences to reach files beyond the allowed directories. The loader does not reject such paths unless the target file is missing and the broken-asset hook is set to `throw`.

### HTML escaping in generated JSX

The loader uses the `escape-html` package to prevent HTML injection in generated JSX attributes and text nodes.

#### `transformImage/index.ts`

- `escapeHtml(node.alt)` for the `alt` attribute
- `escapeHtml(node.title)` for the `title` attribute

#### `transformLinks/index.ts`

- `escapeHtml(node.title)` for the `title` attribute

#### `toc/utils.ts`

- `escapeHtml(node.value)` for text nodes in `toHeadingHTMLValue`
- `escapeHtml(String(classAttribute.value))` for class attributes in `mdxJsxTextElementToHtml`
- `escapeHtml(node.value)` for inline code in `toHeadingHTMLValue`

These calls reduce the risk of injecting arbitrary HTML through Markdown content that becomes part of a heading, image description, or link title.

### JavaScript escaping for `require()` calls

Image and link transformers convert Markdown nodes into JSX attribute value expressions that contain a `require()` call. The `requireString` is built with the `escapePath` utility from `@docusaurus/utils`.

Image:

```ts
const requireString = `${context.inlineMarkdownImageFileLoader}${
  escapePath(relativeImagePath) + search
}`;
```

Link:

```ts
const requireString = `${
  path.extname(relativeAssetPath) === '.json'
    ? `${relativeAssetPath.replace('.json', '.raw')}!=`
    : ''
}${context.inlineMarkdownLinkFileLoader}${
  escapePath(relativeAssetPath) + search
}`;
```

`escapePath` is responsible for escaping file paths so they remain safe inside a JavaScript string literal. The exact implementation is not shown in the provided source, but its use is a control that prevents direct file path injection.

### URL hash and search handling in generated code

`assetRequireAttributeValue` in `remark/utils/index.ts` constructs the generated attribute value:

```ts
value: `require("${requireString}").default${hash && ` + '${hash}'`}`,
```

The `hash` value comes from `parseURLOrPath(node.url).hash`. The `search` value is appended to `requireString` before `escapePath` is applied.

The `hash` is inserted directly into a single-quoted JavaScript string without escaping. If a Markdown image or link URL contains a fragment with a single quote (`'`), backslash, or other JavaScript metacharacters, the generated MDX could contain injection during the build step.

**Threat:** Malicious local Markdown content with a crafted URL fragment could break out of the generated JavaScript string and execute arbitrary code in the build process. This risk is reduced by the trust assumption for source content.

## Secrets management

No hardcoded secrets, API keys, credentials, or sensitive environment variables are present in the provided source.

The only environment variable references are:

- `process.env.NODE_ENV` in `createMDXLoader.ts` for test and production checks
- `process.env.VITEST` in `createMDXLoader.ts`
- `process.versions.pnp` in `transformImage/index.ts` for Yarn PnP detection

None of these expose or manage secrets.

## Threat areas

### 1. Path traversal through Markdown URLs

- **Location:** `transformImage/index.ts`, `transformLinks/index.ts`
- **Cause:** `path.join` is used with decoded user-controlled paths without checking that the result is inside an allowed directory.
- **Impact:** A crafted image or link URL could read arbitrary files if the target exists.

### 2. JavaScript injection through URL fragments and query strings

- **Location:** `remark/utils/index.ts`, `transformImage/index.ts`, `transformLinks/index.ts`
- **Cause:** `hash` is inserted into a generated JavaScript string without escaping; `search` is concatenated before escaping the full string.
- **Impact:** Malicious local content could execute code during the MDX build.

### 3. `target="_blank"` without `rel="noopener"`

In `transformLinks/index.ts`, `toAssetRequireNode` adds:

```ts
attributes.push({
  type: 'mdxJsxAttribute',
  name: 'target',
  value: '_blank',
});
```

No `rel` attribute is added. This opens asset links in a new tab, but the new tab can access the original page’s `window.opener` object.

**Impact:** A malicious destination page could perform reverse tabnabbing against the Docusaurus site. This is a recognized security weakness.

### 4. Unsafe tag name interpolation in TOC serialization

`mdxJsxTextElementToHtml` in `toc/utils.ts` builds an HTML string:

```ts
return `<${tag}${allAttributes}>${content}</${tag}>`;
```

The `tag` value comes from `element.name`, which is derived from MDX JSX content. It is not escaped or validated. Although text content is escaped, an attacker who controls heading structure could inject an arbitrary HTML tag name such as `script` or an event handler attribute through class value handling.

**Note:** This is limited by the trusted author assumption and by `escapeHtml` on text content.

### 5. Broken link policy can hide missing assets

`transformLinks/index.ts` contains this branch for paths with asset-like extensions that are not `@site/`:

```ts
if (hasSiteAlias) {
  node.url = context.onBrokenMarkdownLinks({...}) ?? node.url;
} else {
  // Even if the url has a dot, and it looks like a file extension
  // it can be risky to throw and fail fast by default
}
```

For non-`@site/` paths with extensions like `.pdf` or `.zip`, missing files are silently ignored. This means a broken local asset reference does not trigger the configured broken-link policy unless the `@site/` alias is used. It does not introduce a direct security vulnerability, but it can hide content integrity issues.

## Security controls in code

The following controls are present in the provided source:

- **HTML escaping via `escape-html`:** Used for image alt text, image title, link title, TOC text, inline code, and class attribute values.
- **Path escaping via `escapePath`:** Used for file paths embedded in `require()` call strings.
- **`assetRequireAttributeValue` structured ESTree:** Generates the JavaScript AST for `require()` calls rather than relying on raw string interpolation of the entire expression.
- **`@site/` alias enforcement:** `@site/` paths must exist or trigger a broken asset policy; this forces explicit alias usage for site-local assets.
- **Broken image and link hooks:** Configurable through `siteConfig.markdown.hooks.onBrokenMarkdownImages` and `siteConfig.markdown.hooks.onBrokenMarkdownLinks`. The `throw` value provides fail-fast behavior and is recommended for production.

## Configuration-driven security policies

The MDX loader accepts these options through the Webpack loader configuration:

- `staticDirs: string[]`
- `siteDir: string`
- `onBrokenMarkdownImages: MarkdownConfig['hooks']['onBrokenMarkdownImages']`
- `onBrokenMarkdownLinks: MarkdownConfig['hooks']['onBrokenMarkdownLinks']`

These options are documented in `transformImage/index.ts` and `transformLinks/index.ts`. They map to `siteConfig.markdown.hooks` in the Docusaurus site configuration.

### Recommended policy settings

For the strictest build-time integrity:

```js
// docusaurus.config.js
module.exports = {
  markdown: {
    hooks: {
      onBrokenMarkdownLinks: 'throw',
      onBrokenMarkdownImages: 'throw',
    },
  },
};
```

Using `throw` causes the build to fail when a local image or link cannot be resolved. This can help detect accidental path traversal attempts or missing content.

## Limitations of the provided source

The pre-fetched content does not include full implementations for:

- `packages/docusaurus-mdx-loader/src/remark/contentTitle/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/mermaid/index.ts`
- `packages/docusaurus-mdx-loader/src/remark/admonitions/index.ts`

Those files were listed as directory metadata only. Security analysis for those remark plugins cannot be completed from the provided source.

Additionally, the exact behavior of `escapePath` and `getFileLoaderUtils` from `@docusaurus/utils` is not shown. Their specific escaping and loader construction rules are not documented here.

## Summary

The MDX and Markdown authoring pipeline applies several security controls, especially HTML escaping for generated JSX and path escaping for `require()` calls. However, it does not provide authentication or authorization, and it trusts source content as safe. The most significant threat areas are path traversal from decoded URLs, insufficient escaping of URL fragments in generated JavaScript, missing `rel="noopener"` for asset links, and silent ignoring of some broken local assets.