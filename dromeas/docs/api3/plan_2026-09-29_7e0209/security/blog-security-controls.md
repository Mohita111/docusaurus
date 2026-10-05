# Blog Security Controls

## Scope

This security documentation covers the `@docusaurus/plugin-content-blog` package (`packages/docusaurus-plugin-content-blog/src`) in `Mohita111/docusaurus`, based on the provided source files: `authors.ts`, `blogUtils.ts`, `contentHelpers.ts`, `feed.ts`, `routes.ts`, `index.ts`, `markdownLoader.ts`, and `options.ts`.

## Security models

The blog plugin operates at build time. It reads local Markdown/MDX files, author maps, and tag files, then generates static routes and feed files. There's no runtime server, user database, or session management. Authentication and authorization are outside the plugin.

## Authentication and authorization

- **No authentication:** No authentication logic, login forms, token validation, or credential handling exists in the inspected files.
- **No authorization:** No role-based or attribute-based access control exists. The plugin doesn't distinguish between readers.
- **Trust boundary:** All source files are on the build server's filesystem. Access to publish content is controlled by repository or filesystem permissions, not by the plugin.

## Content visibility controls

### Draft posts

`processBlogSourceFile` in `blogUtils.ts` calls `isDraft({frontMatter})` and, when true, returns `undefined`. The post is skipped from all route generation, lists, and feeds.

```ts
const draft = isDraft({frontMatter});
...
if (draft) {
  return undefined;
}
```

### Unlisted posts

`processBlogSourceFile` also calls `isUnlisted({frontMatter})` and sets `metadata.unlisted`.

`shouldBeListed` in `blogUtils.ts`:

```ts
export function shouldBeListed(blogPost: BlogPost): boolean {
  return !blogPost.metadata.unlisted;
}
```

`routes.ts` uses `shouldBeListed` to filter `listedBlogPosts` for paginated lists, tags, archive, and author pages.

**Important limitation:** `createBlogPostRoutes()` in `routes.ts` maps over the full `blogPosts` array, not `listedBlogPosts`. An unlisted post still has a direct route and can be reached at its permalink if the URL is known or shared. Unlisted isn't an authorization control.

Feeds also exclude unlisted posts via `shouldBeInFeed` in `feed.ts`:

```ts
function shouldBeInFeed(blogPost: BlogPost): boolean {
  const excluded =
    blogPost.metadata.frontMatter.draft ||
    blogPost.metadata.frontMatter.unlisted;
  return !excluded;
}
```

## Data handling and file operations

### Source file reads

- `parseBlogPostMarkdownFile` in `blogUtils.ts` reads blog files with `fs.readFile(filePath, 'utf-8')`.
- `generateBlogPosts` uses `Globby` with the `include` and `exclude` options from plugin configuration to enumerate files under the content path.
- Authors and tags files are read by `getAuthorsMap` and `getTagsFile` imported into `index.ts`.
- Feed generation reads already-generated HTML output using `readOutputHTMLFile` in `feed.ts`.

### Path resolution

`processBlogSourceFile` joins the resolved blog directory with the relative source path:

```ts
const blogSourceAbsolute = path.join(blogDirPath, blogSourceRelative);
```

Because `blogSourceRelative` comes from `Globby` within the content path, path traversal from Markdown file names is constrained.

`feed.ts` resolves XSLT/CSS file paths with `resolveXsltFilePaths`. The `xsltFilePath` option may be absolute or relative. Absolute paths are used as-is. `feedOptions.xslt` must therefore be a trusted configuration value.

`generateXsltFiles` copies the resolved XSLT and CSS files into the output directory using only `path.basename` for output names. No directory traversal occurs in the output filename, but input paths are only checked for existence.

### URL normalization

- `normalizeUrl` from `@docusaurus/utils` is used for permalinks, tag paths, author paths, and feed URLs.
- `normalizeImageUrl` in `authors.ts` prefixes leading `/` image paths with `baseUrl`; external URLs are returned unchanged.
- In feed generation, `defaultCreateFeedItems` converts relative links and image URLs to absolute URLs with:

```ts
const toAbsoluteUrl = (src: string) =>
  String(new URL(src, blogPostAbsoluteUrl));
```

This assumes `src` is a valid URL. A `javascript:` or other scheme would remain if present in generated HTML. Source content is trusted Markdown/MDX.

### Feed generation and HTML handling

- `defaultCreateFeedItems` loads generated HTML with `cheerio`, scopes to `div#${blogPostContainerID}`, and sets feed item content to the HTML of that container.
- The feed files are `rss.xml`, `atom.xml`, and `feed.json`. Output types are validated in `options.ts` as `'rss'`, `'atom'`, `'json'`, `'all'`, or `null`. `null` disables feed output.
- XSLT injection occurs in `injectXslt`:

```ts
return feedContent.replace(
  '<?xml version="1.0" encoding="utf-8"?>',
  `<?xml version="1.0" encoding="utf-8"?><?xml-stylesheet type="text/xsl" href="${path.basename(
    xsltFilePath,
  )}"?>`,
);
```

The inserted value is `path.basename(xsltFilePath)`. It isn't XML-escaped. A basename containing `?>` or quotes could break the feed XML. `xsltFilePath` is a build-time plugin option.

## Secrets and sensitive data

- The inspected plugin code contains no secret management, environment variable access, API keys, tokens, or encryption.
- Don't place secrets in blog content, front matter, `authors.yml`, or tags files. Blog content is compiled into public static files and feed output.
- `authors.yml` may include an `email` field, which `defaultCreateFeedItems` exposes as a feed author email:

```ts
function toFeedAuthor(author: Author): FeedAuthor {
  return {name: author.name, link: author.url, email: author.email};
}
```

Only include author email addresses that are intended for public distribution.

No redaction or encryption is applied to content or metadata at any point.

## Security-relevant configuration

From `options.ts` and plugin defaults:

- `feedOptions.type`: set to `null` to disable RSS/Atom/JSON feed generation. Default is `['rss', 'atom']`.
- `feedOptions.xslt`: custom XSLT/CSS file paths. Only configure trusted local files. Built-in paths are defined in `XSLTBuiltInPaths`.
- `authorsMapPath`: default `'authors.yml'`. Must point to a trusted repository file.
- `tags`: optional path to a tags file. Must be trusted.
- `routeBasePath`, `tagsBasePath`, `archiveBasePath`, `authorsBasePath`, `pageBasePath`: route segments validated by Joi strings. Don't use absolute URLs or path traversal sequences in these settings.
- `onUntruncatedBlogPosts`: values `'ignore'`, `'log'`, `'warn'`, `'throw'`. Not a security control, but `'throw'` can make missing truncation markers fail the build.

## Threat areas and recommendations

| Threat | Present in code | Recommendation |
|---|---|---|
| Malicious JSX in MDX post code | `createBlogMDXLoaderRule` in `index.ts` compiles MDX with plugins; MDX supports React components | Restrict write access to blog source files and review all Markdown/MDX before merge |
| Slug or path manipulation via front matter | `frontMatter.slug ?? parsedBlogFileName.slug` used in `processBlogSourceFile` to build `permalink` | Treat `slug` as trusted content; add validation if accepting untrusted front matter |
| Unlisted post direct URL exposure | `createBlogPostRoutes` uses full `blogPosts` array in `routes.ts` | Don't rely on `unlisted` for access control; treat URLs as public knowledge |
| Feed content injection | `defaultCreateFeedItems` copies HTML into feed items without sanitization | Keep blog content trusted; apply output encoding or consumer-side controls if feeds are consumed by third parties |
| XSLT file path abuse | `resolveXsltFilePaths` accepts absolute paths and copies files | Restrict `feedOptions.xslt` to trusted repository paths; don't expose plugin option modification |
| Feed XML breakage via XSLT basename | `injectXslt` inserts unescaped `path.basename` | Validate `xsltFilePath` basename to prevent XML metacharacters |

## SBOM considerations

- The blog plugin directly imports `feed`, `srcset`, `cheerio`, `lodash`, and `fs-extra` for feed and HTML processing.
- No cryptographic or authentication libraries are present in the provided source files.
- Include these runtime dependencies in any SBOM for `@docusaurus/plugin-content-blog`.