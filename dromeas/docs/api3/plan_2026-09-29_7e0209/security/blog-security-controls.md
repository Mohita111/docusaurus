# Blog Security Controls

This document describes the security controls, data handling behavior, and threat areas for the Docusaurus blog plugin (`@docusaurus/plugin-content-blog`). All statements reference the implementation in `packages/docusaurus-plugin-content-blog/src/`.

## Overview

The blog plugin is a build-time static site generator. It reads Markdown/MDX files from the local filesystem, parses front matter, and emits route metadata, JSON data modules, and optional RSS/Atom/JSON feeds. It does not implement user authentication or runtime authorization. Security-relevant behaviors are concentrated in content visibility filtering, URL normalization, filesystem path resolution, and feed output generation.

## Authentication

The blog plugin does not provide or enforce authentication. Blog posts are static assets generated at build time and served by the hosting layer. There is no login mechanism, session handling, or user identity verification in any of the plugin source files.

Authentication, if required, must be configured at the web server, CDN, or hosting platform level. The plugin only produces static output files through `postBuild` in `packages/docusaurus-plugin-content-blog/src/index.ts`.

## Authorization and content visibility

The plugin enforces two content visibility controls through front matter flags: `draft` and `unlisted`.

### Draft posts

Draft detection uses the `isDraft` utility from `@docusaurus/utils`:

```ts
const draft = isDraft({frontMatter});
```

Source: `processBlogSourceFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

When a post is marked as draft, `processBlogSourceFile` returns `undefined`:

```ts
if (draft) {
  return undefined;
}
```

This excludes the post from route generation, sidebar modules, author pages, tag pages, and all blog listings. Draft posts are never written to the generated output.

### Unlisted posts

Unlisted detection uses the `isUnlisted` utility:

```ts
const unlisted = isUnlisted({frontMatter});
```

Source: `processBlogSourceFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

Unlisted posts are still generated as static pages, but they are filtered out from listing pages using `shouldBeListed`:

```ts
export function shouldBeListed(blogPost: BlogPost): boolean {
  return !blogPost.metadata.unlisted;
}
```

Source: `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

The `shouldBeListed` predicate is applied in `buildAllRoutes` to filter blog listing, tag, author, and archive route inputs. For example:

```ts
const listedBlogPosts = blogPosts.filter(shouldBeListed);
```

Source: `packages/docusaurus-plugin-content-blog/src/routes.ts`.

Unlisted posts are not included in sidebar modules when `blogSidebarCount` is set to `'ALL'` because the sidebar uses `blogPosts` directly. However, the tag visibility logic in `getBlogTags` uses `getTagVisibility` to compute `listedItems` and `unlisted` state separately.

### Feed exclusions

Feed generation applies an additional filter:

```ts
function shouldBeInFeed(blogPost: BlogPost): boolean {
  const excluded =
    blogPost.metadata.frontMatter.draft ||
    blogPost.metadata.frontMatter.unlisted;
  return !excluded;
}
```

Source: `shouldBeInFeed` in `packages/docusaurus-plugin-content-blog/src/feed.ts`.

Draft and unlisted posts are excluded from RSS, Atom, and JSON feeds. This is the only authorization-like control in the feed pipeline.

## Data handling

### File reading

Blog source files are read with `fs.readFile` using UTF-8 encoding:

```ts
const fileContent = await fs.readFile(filePath, 'utf-8');
```

Source: `parseBlogPostMarkdownFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

The file path is constructed from `contentPaths` and validated glob patterns:

```ts
const blogSourceAbsolute = path.join(blogDirPath, blogSourceRelative);
```

Source: `processBlogSourceFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

### Source file enumeration

Source files are enumerated using `Globby` with include and exclude patterns:

```ts
const blogSourceFiles = await Globby(include, {
  cwd: contentPaths.contentPath,
  ignore: exclude,
});
```

Source: `generateBlogPosts` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

The default include pattern is `['**/*.{md,mdx}']`, and the default exclude pattern comes from `GlobExcludeDefault`. Both are validated by `PluginOptionSchema` in `packages/docusaurus-plugin-content-blog/src/options.ts`. File enumeration is constrained to the configured content path.

### Front matter validation

Front matter is validated only after parsing the full file content. The `validateBlogPostFrontMatter` function from `./frontMatter` is invoked inside a try/catch:

```ts
frontMatter: validateBlogPostFrontMatter(result.frontMatter)
```

Source: `parseBlogPostMarkdownFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

The front matter schema is defined in `frontMatter.ts`, which is not included in the provided source files. The validation occurs after file content is read but before metadata is used in route generation.

### URL normalization

All user-derived URLs are normalized through `normalizeUrl` from `@docusaurus/utils`. Permalinks are constructed as:

```ts
const permalink = normalizeUrl([baseUrl, routeBasePath, slug]);
```

Source: `processBlogSourceFile` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

Author image URLs are normalized conditionally based on whether the author has a `key`:

```ts
export function normalizeImageUrl({
  imageURL,
  baseUrl,
}: {
  imageURL: string | undefined;
  baseUrl: string;
}): string | undefined {
  return imageURL?.startsWith('/')
    ? normalizeUrl([baseUrl, imageURL])
    : imageURL;
}
```

Source: `packages/docusaurus-plugin-content-blog/src/authors.ts`.

Global authors with keys throw an error if their image URL starts with `/` but is not already prefixed with the base URL:

```ts
if (
  author.imageURL?.startsWith('/') &&
  !author.imageURL.startsWith(baseUrl)
) {
  throw new Error(
    `Docusaurus internal bug: global authors image ${author.imageURL} should start with the expected baseUrl=${baseUrl}`,
  );
}
```

Source: `normalizeAuthorUrl` in `packages/docusaurus-plugin-content-blog/src/authors.ts`.

This invariant prevents inconsistent absolute image paths for authors loaded from `authors.yml`.

### Authors map validation

Author keys referenced in front matter are resolved against the authors map. Missing maps or missing keys throw errors:

```ts
if (!authorsMap || Object.keys(authorsMap).length === 0) {
  throw new Error(`Can't reference blog post authors by a key ...`);
}
```

```ts
if (!author) {
  throw Error(`Blog author with key "${key}" not found ...`);
}
```

Source: `getAuthorsMapAuthor` in `packages/docusaurus-plugin-content-blog/src/authors.ts`.

Parse errors in any source file cause the entire build to fail with a wrapped error:

```ts
throw new Error(
  `Processing of blog source file path=${blogSourceFile} failed.`,
  {cause: err},
);
```

Source: `generateBlogPosts` in `packages/docusaurus-plugin-content-blog/src/blogUtils.ts`.

## Secrets management

The blog plugin has no runtime secret storage or retrieval mechanism. It does not read environment variables for secrets, access external secret managers, or embed credentials in generated output.

The only configuration file loaded by the plugin is `authors.yml` and the optional tags file. These are read through `getAuthorsMap` and `getTagsFile`. The files are sourced from the local content paths:

```ts
const authorsMapFilePath = await getDataFilePath({
  filePath: options.authorsMapPath,
  contentPaths,
});
```

Source: `pluginContentBlog` in `packages/docusaurus-plugin-content-blog/src/index.ts`.

Important: this is a build-time static generator. Any secrets placed in front matter, `authors.yml`, or Markdown/MDX content are copied into the generated JSON metadata modules and the static HTML output. The plugin does not redact content. Sensitive values must never be placed in blog source files or front matter.

## Feed security controls

### HTML content handling in feeds

Feed items are built by parsing the post HTML with Cheerio and extracting the content inside `div#${blogPostContainerID}`:

```ts
const content = await readOutputHTMLFile(
  permalink.replace(baseUrl, ''),
  outDir,
  trailingSlash,
);
const $ = cheerioLoad(content);

// ...
content: $(`#${blogPostContainerID}`).html()!,
```

Source: `defaultCreateFeedItems` in `packages/docusaurus-plugin-content-blog/src/feed.ts`.

The plugin does not sanitize the extracted HTML. Feed content is emitted as-is from the generated post HTML. If a Markdown/MDX pipeline permits raw HTML, arbitrary HTML included by a blog post author is passed into the RSS/Atom/JSON feed output.

### URL absolutization

Feed generation converts relative links and image URLs to absolute URLs before emitting feed items:

```ts
const toAbsoluteUrl = (src: string) =>
  String(new URL(src, blogPostAbsoluteUrl));

$(`div#${blogPostContainerID} a, div#${blogPostContainerID} img`).each(
  (_, elm) => {
    if (elm.tagName === 'a') {
      const {href} = elm.attribs;
      if (href) {
        elm.attribs.href = toAbsoluteUrl(href);
      }
    } else if (elm.tagName === 'img') {
      const {src, srcset: srcsetAttr} = elm.attribs;
      if (src) {
        elm.attribs.src = toAbsoluteUrl(src);
      }
      if (srcsetAttr) {
        elm.attribs.srcset = stringifySrcset(
          parseSrcset(srcsetAttr).map((props) => ({
            ...props,
            url: toAbsoluteUrl(props.url),
          })),
        );
      }
    }
  },
);
```

Source: `defaultCreateFeedItems` in `packages/docusaurus-plugin-content-blog/src/feed.ts`.

Both `href` and `src`/`srcset` values are passed to `new URL()`. Malformed or protocol-relative URLs are resolved against the post permalink, which is derived from `normalizeUrl`.

### XSLT file resolution

XSLT paths are resolved from either an absolute path or the content path:

```ts
const xsltAbsolutePath: string = path.isAbsolute(xsltFilePath)
  ? xsltFilePath
  : ((await getDataFilePath({filePath: xsltFilePath, contentPaths})) ??
    path.resolve(contentPaths.contentPath, xsltFilePath));
```

Source: `resolveXsltFilePaths` in `packages/docusaurus-plugin-content-blog/src/feed.ts`.

The XSLT option is validated against `createXSLTFilePathSchema`, which accepts `Joi.string().required()` or a boolean. If a user supplies an absolute path outside the content directory, the plugin reads it directly. XSLT files are copied verbatim to the output directory without transformation. A co-located CSS file with the same basename is required and is validated with `fs.pathExists`.

### XSLT injection

The XSLT reference is injected into the XML feed content through string replacement:

```ts
return feedContent.replace(
  '<?xml version="1.0" encoding="utf-8"?>',
  `<?xml version="1.0" encoding="utf-8"?><?xml-stylesheet type="text/xsl" href="${path.basename(
    xsltFilePath,
  )}"?>`,
);
```

Source: `injectXslt` in `packages/docusaurus-plugin-content-blog/src/feed.ts`.

`path.basename` prevents path traversal in the emitted `href`. The XSLT file content itself is not validated beyond the co-located CSS existence check.

## Threat areas

### Path traversal in XSLT configuration

`resolveXsltFilePaths` accepts an absolute `xsltFilePath` string directly. If an administrator configures a path that points outside the intended content directory, the plugin reads and copies that file during build. This is only exploitable at configuration time by a developer with file system access, not by site visitors. Mitigation: use built-in XSLT paths or place custom XSLT files under the configured content path.

### Path traversal through edit URL

The `editUrl` option accepts either a string or a function:

```ts
editUrl: Joi.alternatives().try(URISchema, Joi.function()),
```

Source: `PluginOptionSchema` in `packages/docusaurus-plugin-content-blog/src/options.ts`.

String edit URLs are normalized and combined with source-relative paths via `getEditUrl`. Function-based edit URLs receive `blogDirPath`, `blogPath`, and `permalink` as input. The function return value is used directly in metadata without additional URL validation. A malicious or misconfigured `editUrl` function could inject arbitrary strings into generated metadata modules.

### Feed content injection

The feed pipeline does not sanitize HTML before inserting it into RSS/Atom/JSON feeds. Blog post content originates from Markdown/MDX files authored by trusted developers, but if the content pipeline allows raw HTML or imports from external sources, feed subscribers may encounter active content. RSS readers often sanitize HTML, but JSON feed consumers may not.

### Author URL and image URL normalization

Author URLs from front matter and `authors.yml` are normalized only when they start with `/`. Full URLs such as `https://example.com` or protocol-relative URLs are passed through unchanged:

```ts
return imageURL?.startsWith('/')
  ? normalizeUrl([baseUrl, imageURL])
  : imageURL;
```

Source: `normalizeImageUrl` in `packages/docusaurus-plugin-content-blog/src/authors.ts`.

An author entry can set `url` to an arbitrary external site and `imageURL` to an arbitrary external image URL. These are emitted into author metadata and feed items. An author with write access to a blog post can cause generated pages to reference arbitrary external URLs.

### Author key confusion

When a front matter `authors` value is a string, it is treated as a key and not a display name:

```ts
if (typeof authorInput === 'string') {
  return {key: authorInput};
}
```

Source: `normalizeFrontMatterAuthors` in `packages/docusaurus-plugin-content-blog/src/authors.ts`.

A typo in an author key causes a hard build error rather than silently falling back to a display name. This reduces the chance of misattributing posts to an unintended author.

### Legacy author mixing

Mixing legacy author front matter fields (`author`, `author_title`, `author_url`) with the modern `authors` field causes a build error:

```ts
if (authors.length > 0) {
  throw new Error(
    `To declare blog post authors, use the 'authors' front matter in priority. ...`,
  );
}
```

Source: `getBlogPostAuthors` in `packages/docusaurus-plugin-content-blog/src/authors.ts`.

## Configuration guidance

The following security-relevant plugin options are validated by `PluginOptionSchema` in `packages/docusaurus-plugin-content-blog/src/options.ts`.

| Option | Type | Default | Security notes |
|--------|------|---------|----------------|
| `path` | string | `'blog'` | Root directory for blog content. Resolved against `siteDir`. |
| `include` | array of glob strings | `['**/*.{md,mdx}']` | Controls which files are read as blog posts. |
| `exclude` | array of glob strings | `GlobExcludeDefault` | Controls which files are ignored by `Globby`. |
| `authorsMapPath` | string | `'authors.yml'` | Path to the authors map. Resolved through `getDataFilePath`. |
| `feedOptions.type` | `'rss'`, `'atom'`, `'json'`, `'all'`, array, or `null` | `['rss','atom']` | Controls generated feed types. Set to `null` to disable feeds. |
| `feedOptions.xslt` | object, boolean, or `null` | `{rss: null, atom: null}` | Controls XSLT files copied and referenced by XML feeds. Accepts file paths. |
| `editUrl` | string or function | undefined | Controls **Edit this page** links emitted into metadata. |
| `onInlineTags` | `'ignore'`, `'log'`, `'warn'`, `'throw'` | `'warn'` | Controls handling of inline tags. |
| `onInlineAuthors` | `'ignore'`, `'log'`, `'warn'`, `'throw'` | `'warn'` | Controls handling of inline authors. |
| `onUntruncatedBlogPosts` | `'ignore'`, `'log'`, `'warn'`, `'throw'` | `'warn'` | Controls reporting of posts without truncation markers. |

## Security boundaries

The blog plugin operates entirely at build time. Its trust boundary is the local filesystem and the developers who author blog Markdown/MDX files and configure the plugin. There is no runtime input validation because there is no runtime request handling.

Untrusted content must be handled by the deployment environment. The plugin's output includes static HTML, JSON metadata modules, and feed files. Any content authored in blog posts becomes part of these outputs without sanitization by the blog plugin itself.

## Relevant source files

- `packages/docusaurus-plugin-content-blog/src/authors.ts` — author normalization and resolution
- `packages/docusaurus-plugin-content-blog/src/blogUtils.ts` — file reading, draft/unlisted handling, date parsing, pagination
- `packages/docusaurus-plugin-content-blog/src/feed.ts` — feed generation, XSLT handling, HTML content extraction
- `packages/docusaurus-plugin-content-blog/src/routes.ts` — route generation and content visibility filtering
- `packages/docusaurus-plugin-content-blog/src/index.ts` — plugin lifecycle, content loading, webpack configuration
- `packages/docusaurus-plugin-content-blog/src/options.ts` — option schema and default values
- `packages/docusaurus-plugin-content-blog/src/markdownLoader.ts` — Markdown truncation for list previews
- `packages/docusaurus-plugin-content-blog/src/contentHelpers.ts` — source-to-permalink mapping used by MDX loader