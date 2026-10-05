# Blog internals

This document describes the implementation of`@docusaurus/plugin-content-blog`, located in`packages/docusaurus-plugin-content-blog/src`. It covers the modules, data flow, route generation, MDX loader, feed generation, and extension points behind Docusaurus blogs.

## Architecture at a glance

The blog plugin is a single default export from`src/index.ts`:

```ts
export default async function pluginContentBlog(
  context: LoadContext,
  options: PluginOptions,
): Promise<Plugin<BlogContent>>
```

The plugin lifecycle is split across these phases:

1. **Option validation** in`src/options.ts`
2. **Content loading** in`src/index.ts` and`src/blogUtils.ts`
3. **Route generation** in`src/routes.ts`
4. **Webpack configuration** in`src/index.ts` and`src/markdownLoader.ts`
5. **Feed generation** in`src/feed.ts`
6. **Translation** in`src/translations.ts`

The following diagram shows how the modules interconnect:

```mermaid
graph TD
    index["src/index.ts<br/>Plugin entry point"] --> options["src/options.ts<br/>Joi validation"]
    index --> blogUtils["src/blogUtils.ts<br/>Content processing"]
    index --> routes["src/routes.ts<br/>Route generation"]
    index --> feed["src/feed.ts<br/>Feed generation"]
    index --> authors["src/authors.ts<br/>Author resolution"]
    index --> contentHelpers["src/contentHelpers.ts<br/>MDX loader maps"]
    index --> markdownLoader["src/markdownLoader.ts<br/>Truncation"]
    blogUtils --> authors
    routes --> blogUtils
    routes --> authors
    feed --> authors
```

### Package layout

| File | Responsibility |
| --- | --- |
|`src/index.ts` | Plugin entry point, lifecycle methods, MDX loader setup |
|`src/blogUtils.ts` | Source file processing, date handling, pagination, tag aggregation |
|`src/authors.ts` | Author resolution and grouping |
|`src/routes.ts` | Route configuration generation |
|`src/feed.ts` | RSS, Atom, and JSON feed generation |
|`src/contentHelpers.ts` | Mutable maps used by the MDX loader |
|`src/markdownLoader.ts` | Webpack loader for truncating MDX content |
|`src/options.ts` | Default options and Joi validation |

## Plugin lifecycle

The following sequence diagram shows how Docusaurus core drives the plugin lifecycle from content loading through build:

```mermaid
sequenceDiagram
    participant Core as Docusaurus core
    participant PI as pluginContentBlog
    participant BU as blogUtils
    participant FE as feed
    
    Core->>PI: validateOptions(options)
    Note over PI: Joi validation in options.ts
    Core->>PI: loadContent()
    PI->>PI: getAuthorsMapChecked()
    PI->>PI: getTagsFile()
    PI->>BU: generateBlogPosts(contentPaths, context, options, tagsFile, authorsMap)
    BU-->>PI: BlogPost[]
    PI->>PI: applyProcessBlogPosts()
    PI->>BU: getBlogTags(...)
    PI-->>Core: BlogContent
    Core->>PI: contentLoaded(content, actions)
    PI->>PI: contentHelpers.updateContent(content)
    PI->>PI: createAllRoutes(...)
    PI-->>Core: routes registered
    Core->>PI: postBuild({outDir, content})
    PI->>FE: createBlogFeedFiles(...)
    FE-->>PI: feed files written
```

### Hash router limitation

`src/index.ts` disables feed generation when the experimental hash router is configured:

```ts
const router = siteConfig.future.experimental_router;
const isBlogFeedDisabledBecauseOfHashRouter =
  router === 'hash' && !!options.feedOptions.type;
if (isBlogFeedDisabledBecauseOfHashRouter) {
  logger.warn(
    `${PluginName} feed feature does not support the Hash Router. Feeds won't be generated.`,
  );
}
```

The`isBlogFeedDisabledBecauseOfHashRouter` guard is checked in both`postBuild` and`injectHtmlTags`.

###`getPathsToWatch`

The plugin watches author map files, tag files, and all blog source files matching the configured`include` patterns:

```ts
getPathsToWatch() {
  const {include} = options;
  const contentMarkdownGlobs = getContentPathList(contentPaths).flatMap(
    (contentPath) => include.map((pattern) => `${contentPath}/${pattern}`),
  );

  const tagsFilePaths = getTagsFilePathsToWatch({
    contentPaths,
    tags: options.tags,
  });

  return [
    authorsMapFilePath,
    ...tagsFilePaths,
    ...contentMarkdownGlobs,
  ].filter(Boolean) as string[];
}
```

###`getTranslationFiles`

Delegates to`getTranslationFiles(options)` from`src/translations.ts`.

###`translateContent`

Applies translations to loaded content:

```ts
translateContent({content, translationFiles}) {
  return translateContent(content, translationFiles);
}
```

## Data flow

The main data flow starts in`src/index.ts`.

###`loadContent`

```ts
const {authorsMap, tagsFile} = await combinePromises({
  authorsMap: getAuthorsMapChecked(),
  tagsFile: getTagsFile({contentPaths, tags: options.tags}),
});

let blogPosts = await generateBlogPosts(
  contentPaths,
  context,
  options,
  tagsFile,
  authorsMap,
);
blogPosts = await applyProcessBlogPosts({
  blogPosts,
  processBlogPosts: options.processBlogPosts,
});
```

After generating`blogPosts`, the plugin:

1. Calls`reportUntruncatedBlogPosts` to warn about missing truncation markers.
2. Filters unlisted posts with`shouldBeListed`.
3. Returns an empty`BlogContent` if no posts exist.
4. Computes`prevItem` and`nextItem` metadata for listed posts:

```ts
listedBlogPosts.forEach((blogPost, index) => {
  const prevItem = index > 0 ? listedBlogPosts[index - 1] : null;
  if (prevItem) {
    blogPost.metadata.prevItem = {
      title: prevItem.metadata.title,
      permalink: prevItem.metadata.permalink,
    };
  }

  const nextItem =
    index < listedBlogPosts.length - 1
      ? listedBlogPosts[index + 1]
      : null;
  if (nextItem) {
    blogPost.metadata.nextItem = {
      title: nextItem.metadata.title,
      permalink: nextItem.metadata.permalink,
    };
  }
});
```

5. Aggregates tags with`getBlogTags`:

```ts
const blogTags: BlogTags = getBlogTags({
  blogPosts,
  postsPerPageOption,
  blogDescription,
  blogTitle,
  pageBasePath,
});
```

6. Returns the`BlogContent` object:

```ts
return {
  blogTitle,
  blogDescription,
  blogSidebarTitle,
  blogPosts,
  blogTags,
  blogTagsListPath,
  authorsMap,
};
```

###`contentLoaded`

```ts
async contentLoaded({content, actions}) {
  contentHelpers.updateContent(content);
  await createAllRoutes({
    baseUrl,
    content,
    actions,
    options,
    aliasedSource,
  });
}
```

`updateContent` populates the source-to-post and source-to-permalink maps before routes are built. The`aliasedSource` function creates virtual module paths rooted at`~blog`:

```ts
const aliasedSource = (source: string) =>
  `~blog/${posixPath(path.relative(pluginDataDirRoot, source))}`;
```

###`configureWebpack`

The webpack configuration:

- Registers the`~blog` alias:

```ts
resolve: {
  alias: {
    '~blog': pluginDataDirRoot,
  },
},
```

- Adds an MDX loader rule for`.md` and`.mdx` files inside the blog content directories. The rule applies both the standard MDX loader and a custom`markdownLoader` that handles truncation.

###`postBuild`

If the blog has posts and feed options are enabled,`postBuild` calls`createBlogFeedFiles`:

```ts
async postBuild({outDir, content}) {
  if (
    !content.blogPosts.length ||
    !options.feedOptions.type ||
    isBlogFeedDisabledBecauseOfHashRouter
  ) {
    return;
  }

  await createBlogFeedFiles({
    blogPosts: content.blogPosts,
    options,
    outDir,
    siteConfig,
    locale: currentLocale,
    contentPaths,
  });
}
```

###`injectHtmlTags`

If feeds are enabled,`injectHtmlTags` exposes feed link tags through`createFeedHtmlHeadTags`:

```ts
injectHtmlTags({content}) {
  if (
    !content.blogPosts.length ||
    !options.feedOptions.type ||
    isBlogFeedDisabledBecauseOfHashRouter
  ) {
    return {};
  }

  return {headTags: createFeedHtmlHeadTags({context, options})};
}
```

## Blog source processing

The core source processing logic lives in`src/blogUtils.ts`.

###`generateBlogPosts`

Signature:

```ts
export async function generateBlogPosts(
  contentPaths: BlogContentPaths,
  context: LoadContext,
  options: PluginOptions,
  tagsFile: TagsFile | null,
  authorsMap?: AuthorsMap,
): Promise<BlogPost[]>
```

Algorithm:

1. If`contentPaths.contentPath` does not exist, return`[]`:

```ts
if (!(await fs.pathExists(contentPaths.contentPath))) {
  return [];
}
```

2. Use`Globby(include, {cwd: contentPaths.contentPath, ignore: exclude})` to find source files.

3. Process each file in parallel with`Promise.all` through`doProcessBlogSourceFile`, which wraps`processBlogSourceFile` with an error message that includes the failed file path:

```ts
async function doProcessBlogSourceFile(blogSourceFile: string) {
  try {
    return await processBlogSourceFile(
      blogSourceFile,
      contentPaths,
      context,
      options,
      tagsFile,
      authorsMap,
    );
  } catch (err) {
    throw new Error(
      `Processing of blog source file path=${blogSourceFile} failed.`,
      {cause: err},
    );
  }
}
```

4. Filter out`undefined` values returned for draft posts:

```ts
const blogPosts = (
  await Promise.all(blogSourceFiles.map(doProcessBlogSourceFile))
).filter(Boolean) as BlogPost[];
```

5. Sort descending by`metadata.date`:

```ts
blogPosts.sort(
  (a, b) => b.metadata.date.getTime() - a.metadata.date.getTime(),
);
```

6. Reverse if`options.sortPosts === 'ascending'`:

```ts
if (options.sortPosts === 'ascending') {
  return blogPosts.reverse();
}
return blogPosts;
```

###`processBlogSourceFile`

This private function transforms a single source file into a`BlogPost`.

Steps:

1. Resolve the localized content folder with`getFolderContainingFile`:

```ts
const blogDirPath = await getFolderContainingFile(
  getContentPathList(contentPaths),
  blogSourceRelative,
);
```

2. Join the source directory and relative file name:

```ts
const blogSourceAbsolute = path.join(blogDirPath, blogSourceRelative);
```

3. Parse the markdown file with`parseBlogPostMarkdownFile`, which reads the file, calls`parseMarkdownFile` with`removeContentTitle: true`, validates front matter through`validateBlogPostFrontMatter`, and logs an error with the file path on failure.

4. Compute`aliasedSource` with`aliasedSitePath`:

```ts
const aliasedSource = aliasedSitePath(blogSourceAbsolute, siteDir);
```

5. Read last update data with`readLastUpdateData`:

```ts
const lastUpdate = await readLastUpdateData(
  blogSourceAbsolute,
  options,
  frontMatter.last_update,
  vcs,
);
```

6. Check draft and unlisted status with`isDraft` and`isUnlisted`:

```ts
const draft = isDraft({frontMatter});
const unlisted = isUnlisted({frontMatter});

if (draft) {
  return undefined;
}
```

7. Warn about the deprecated`id` front matter field:

```ts
if (frontMatter.id) {
  logger.warn`name=${'id'} header option is deprecated in path=${blogSourceRelative} file. Please use name=${'slug'} option instead.`;
}
```

8. Parse the file name date and slug with`parseBlogFileName`:

```ts
const parsedBlogFileName = parseBlogFileName(blogSourceRelative);
```

9. Resolve the final date with`getBlogPostDate`:

```ts
const date = await getBlogPostDate({
  frontMatterDate: frontMatter.date,
  filenameDate: parsedBlogFileName.date,
  filePath: blogSourceAbsolute,
  vcs,
});
```

10. Compute title with a three-tier fallback:

```ts
const title = frontMatter.title ?? contentTitle ?? parsedBlogFileName.text;
```

11. Compute description:

```ts
const description = frontMatter.description ?? excerpt ?? '';
```

12. Compute slug:

```ts
const slug = frontMatter.slug ?? parsedBlogFileName.slug;
```

13. Compute permalink:

```ts
const permalink = normalizeUrl([baseUrl, routeBasePath, slug]);
```

14. Compute tags base route path:

```ts
const tagsBaseRoutePath = normalizeUrl([
  baseUrl,
  routeBasePath,
  tagsRouteBasePath,
]);
```

15. Resolve authors with`getBlogPostAuthors` and check for author problems with`reportAuthorsProblems`.

16. Resolve tags with`normalizeTags`:

```ts
const tags = normalizeTags({
  options,
  source: blogSourceRelative,
  frontMatterTags: frontMatter.tags,
  tagsBaseRoutePath,
  tagsFile,
});
```

17. Return the completed`BlogPost` object:

```ts
return {
  id: slug,
  metadata: {
    permalink,
    editUrl: getBlogEditUrl(),
    source: aliasedSource,
    title,
    description,
    date,
    tags,
    readingTime: showReadingTime
      ? options.readingTime({
          content,
          frontMatter,
          defaultReadingTime,
          locale: i18n.currentLocale,
        })
      : undefined,
    hasTruncateMarker: truncateMarker.test(content),
    authors,
    frontMatter,
    unlisted,
    lastUpdatedAt: lastUpdate.lastUpdatedAt,
    lastUpdatedBy: lastUpdate.lastUpdatedBy,
  },
  content,
};
```

###`getBlogEditUrl`

The edit URL supports both string and function forms. With a function, the plugin passes`blogDirPath`,`blogPath`,`permalink`, and`locale`:

```ts
if (typeof editUrl === 'function') {
  return editUrl({
    blogDirPath: posixPath(path.relative(siteDir, blogDirPath)),
    blogPath: posixPath(blogPathRelative),
    permalink,
    locale: i18n.currentLocale,
  });
}
```

With a string, the plugin constructs the URL by normalizing the edit URL with the relative file content path:

```ts
if (typeof editUrl === 'string') {
  const isLocalized = blogDirPath === contentPaths.contentPathLocalized;
  const fileContentPath =
    isLocalized &&
    options.editLocalizedFiles &&
    contentPaths.contentPathLocalized
      ? contentPaths.contentPathLocalized
      : contentPaths.contentPath;

  const contentPathEditUrl = normalizeUrl([
    editUrl,
    posixPath(path.relative(siteDir, fileContentPath)),
  ]);

  return getEditUrl(blogPathRelative, contentPathEditUrl);
}
return undefined;
```

###`parseBlogFileName`

The date extraction regex is:

```ts
const DATE_FILENAME_REGEX =
  /^(?<folder>.*)(?<date>\d{4}[-/]\d{1,2}[-/]\d{1,2})[-/]?(?<text>.*?)(?:\/index)?.mdx?$/;
```

If the file name contains a date in`YYYY-MM-DD` or`YYYY/M/D` format, the date is parsed as UTC by appending`Z`:

```ts
const date = new Date(`${dateString!}Z`);
```

The slug is built from folder and remaining text:

```ts
const slugDate = dateString!.replace(/-/g, '/');
const slug = `/${slugDate}/${folder!}${text!}`;
```

For file names without a date, the slug is derived from the file name minus`.md`/`.mdx` and optional`/index`:

```ts
const text = blogSourceRelative.replace(/(?:\/index)?\.mdx?$/, '');
const slug = `/${text}`;
```

Returns a`ParsedBlogFileName` object with`date`,`text`, and`slug` properties.

###`getBlogPostDate`

Date resolution follows a priority order:

1. Front matter`date`, normalized with`parseFrontMatterDate`
2. File name date extracted from`parseBlogFileName`
3. VCS file creation info via`vcs.getFileCreationInfo(filePath)`
4. File system`birthtime` as fallback

```ts
if (frontMatterDate) {
  return parseFrontMatterDate(frontMatterDate);
} else if (filenameDate) {
  return filenameDate;
}

const result = await vcs.getFileCreationInfo(filePath);
if (result == null) {
  return (await fs.stat(filePath)).birthtime;
}
return new Date(result.timestamp);
```

###`parseFrontMatterDate`

String dates without timezone are treated as UTC by appending`Z`. Dates with timezone suffixes are parsed as-is. The timezone detection regex is:

```ts
const hasTimeZone = /[t \t]\d.*(?:z|[+-]\d{1,2}(?::?\d{2})?)$/i.test(
  dateString,
);
return new Date(hasTimeZone ? dateString : `${dateString}Z`);
```

Follows YAML timestamp conventions and accepts ISO offsets like`+05:30` or`-0800`.

###`applyProcessBlogPosts`

This function applies the`processBlogPosts` extension point:

```ts
export async function applyProcessBlogPosts({
  blogPosts,
  processBlogPosts,
}: {
  blogPosts: BlogPost[];
  processBlogPosts: PluginOptions['processBlogPosts'];
}): Promise<BlogPost[]> {
  const processedBlogPosts = await processBlogPosts({blogPosts});

  if (Array.isArray(processedBlogPosts)) {
    return processedBlogPosts;
  }

  return blogPosts;
}
```

If`processBlogPosts` returns an array, that array becomes the new blog posts list. Otherwise, the original list is preserved.

###`reportUntruncatedBlogPosts`

Warns about blog posts missing truncation markers unless`onUntruncatedBlogPosts: 'ignore'` is configured. The function checks for the truncation marker with`truncateMarker.test(content)` and emits a warning via`logger.report`:

```ts
export function reportUntruncatedBlogPosts({
  blogPosts,
  onUntruncatedBlogPosts,
}: {
  blogPosts: BlogPost[];
  onUntruncatedBlogPosts: PluginOptions['onUntruncatedBlogPosts'];
}): void {
  const untruncatedBlogPosts = blogPosts.filter(
    (p) => !p.metadata.hasTruncateMarker,
  );
  if (onUntruncatedBlogPosts !== 'ignore' && untruncatedBlogPosts.length > 0) {
    const message = logger.interpolate`Docusaurus found blog posts without truncation markers:
- ${untruncatedBlogPosts
      .map((p) => logger.path(aliasedSitePathToRelativePath(p.metadata.source)))
      .join('\n- ')}

We recommend using truncation markers (code=${`<!-- truncate -->`} or code=${`{/* truncate */}`}) in blog posts to create shorter previews on blog paginated lists.
Tip: turn this security off with the code=${`onUntruncatedBlogPosts: 'ignore'`} blog plugin option.`;
    logger.report(onUntruncatedBlogPosts)(message);
  }
}
```

## Pagination and tags

###`paginateBlogPosts`

Signature:

```ts
export function paginateBlogPosts({
  blogPosts,
  basePageUrl,
  blogTitle,
  blogDescription,
  postsPerPageOption,
  pageBasePath,
}: {
  blogPosts: BlogPost[];
  basePageUrl: string;
  blogTitle: string;
  blogDescription: string;
  postsPerPageOption: number | 'ALL';
  pageBasePath: string;
}): BlogPaginated[]
```

Behavior:

-`postsPerPageOption === 'ALL'` uses all posts on one page:

```ts
const postsPerPage =
  postsPerPageOption === 'ALL' ? totalCount : postsPerPageOption;
```

- Page count is at least 1:

```ts
const numberOfPages = Math.max(1, Math.ceil(totalCount / postsPerPage));
```

- Page 0 uses`basePageUrl`
- Pages greater than 0 use:

```ts
normalizeUrl([basePageUrl, pageBasePath, `${page + 1}`])
```

- Each page receives`previousPage` and`nextPage` permalinks:

```ts
metadata: {
  permalink: permalink(page),
  page: page + 1,
  postsPerPage,
  totalPages: numberOfPages,
  totalCount,
  previousPage: page !== 0 ? permalink(page - 1) : undefined,
  nextPage: page < numberOfPages - 1 ? permalink(page + 1) : undefined,
  blogDescription,
  blogTitle,
},
```

###`shouldBeListed`

A blog post is listed if it is not unlisted:

```ts
export function shouldBeListed(blogPost: BlogPost): boolean {
  return !blogPost.metadata.unlisted;
}
```

###`getBlogTags`

Tags are grouped with`groupTaggedItems`, filtered by visibility, paginated, and returned as`BlogTags`:

```ts
const groups = groupTaggedItems(
  blogPosts,
  (blogPost) => blogPost.metadata.tags,
);
return _.mapValues(groups, ({tag, items: tagBlogPosts}) => {
  const tagVisibility = getTagVisibility({
    items: tagBlogPosts,
    isUnlisted: (item) => item.metadata.unlisted,
  });
  return {
    inline: tag.inline,
    label: tag.label,
    permalink: tag.permalink,
    description: tag.description,
    items: tagVisibility.listedItems.map((item) => item.id),
    pages: paginateBlogPosts({
      blogPosts: tagVisibility.listedItems,
      basePageUrl: tag.permalink,
      ...params,
    }),
    unlisted: tagVisibility.unlisted,
  };
});
```

Each tag result includes the tag's inline status, label, permalink, description, listed post IDs, paginated tag pages, and unlisted status.

## Author resolution

Author logic is in`src/authors.ts`.

###`getBlogPostAuthors`

Signature:

```ts
export function getBlogPostAuthors(params: AuthorsParam): Author[]
```

`AuthorsParam` contains:

```ts
type AuthorsParam = {
  frontMatter: BlogPostFrontMatter;
  authorsMap: AuthorsMap | undefined;
  baseUrl: string;
};
```

Resolution behavior:

1. Legacy front matter fields are checked first (`author`,`author_title`,`authorTitle`,`author_url`,`authorURL`,`author_image_url`,`authorImageURL`). If any of these are present, they are assembled into an`Author` object with`key: null` and`page: null`.

2. If legacy fields exist and modern`frontMatter.authors` also exists, an error is thrown:

```ts
if (authorLegacy) {
  if (authors.length > 0) {
    throw new Error(
      `To declare blog post authors, use the 'authors' front matter in priority.
Don't mix 'authors' with other existing 'author_*' front matter. Choose one or the other, not both at the same time.`,
    );
  }
  return [authorLegacy];
}
```

3. Otherwise,`frontMatter.authors` is normalized. Authors can be specified as a string (author key), an object, or an array of either:

```ts
function normalizeAuthor(
  authorInput: string | BlogPostFrontMatterAuthor,
): BlogPostFrontMatterAuthor {
  if (typeof authorInput === 'string') {
    return {key: authorInput};
  }
  return {
    ...authorInput,
    socials: normalizeSocials(authorInput.socials ?? {}),
  };
}

return Array.isArray(frontMatter.authors)
  ? frontMatter.authors.map(normalizeAuthor)
  : [normalizeAuthor(frontMatter.authors)];
```

4. For each normalized author, if a`key` is provided, the author is looked up in`authorsMap`. If the map is missing or the key doesn't exist, an error is thrown:

```ts
const author = authorsMap[key];
if (!author) {
  throw Error(`Blog author with key "${key}" not found in the authors map file.
Valid author keys are:
${Object.keys(authorsMap)
  .map((validKey) => `- ${validKey}`)
  .join('\n')}`);
}
```

5. Global author data from the map is merged with front matter overrides:

```ts
const author = {
  ...getAuthorsMapAuthor(frontMatterAuthor.key),
  ...frontMatterAuthor,
} as Author;
```

6. Author image URLs are normalized with`normalizeAuthorUrl`, which preserves URLs from the map (already normalized) and normalizes relative URLs from inline front matter:

```ts
function normalizeAuthorUrl({
  author,
  baseUrl,
}: {
  author: Author;
  baseUrl: string;
}): string | undefined {
  if (author.key) {
    if (
      author.imageURL?.startsWith('/') &&
      !author.imageURL.startsWith(baseUrl)
    ) {
      throw new Error(
        `Docusaurus internal bug: global authors image ${author.imageURL} should start with the expected baseUrl=${baseUrl}`,
      );
    }
    return author.imageURL;
  }
  return normalizeImageUrl({imageURL: author.imageURL, baseUrl});
}
```

###`groupBlogPostsByAuthorKey`

Groups blog posts by author key, excluding posts with only inline (non-keyed) authors:

```ts
export function groupBlogPostsByAuthorKey({
  blogPosts,
  authorsMap,
}: {
  blogPosts: BlogPost[];
  authorsMap: AuthorsMap | undefined;
}): Record<string, BlogPost[]> {
  return _.mapValues(authorsMap, (author, key) =>
    blogPosts.filter((p) => p.metadata.authors.some((a) => a.key === key)),
  );
}
```

## Route generation

Route logic is in`src/routes.ts`.

###`buildAllRoutes`

Signature:

```ts
export async function buildAllRoutes({
  baseUrl,
  content,
  actions,
  options,
  aliasedSource,
}: CreateAllRoutesParam): Promise<RouteConfig[]>
```

This function generates all blog routes: individual blog post routes, paginated blog listing routes, tag routes, author routes, and an optional archive route.

The diagram below shows the major route groups:

```mermaid
graph TD
    buildAllRoutes["buildAllRoutes()"]
    
    buildAllRoutes --> posts["createBlogPostRoutes()"]
    buildAllRoutes --> paginated["createBlogPostsPaginatedRoutes()"]
    buildAllRoutes --> tags["createTagsRoutes()"]
    buildAllRoutes --> archive["createArchiveRoute()"]
    buildAllRoutes --> authors["createAuthorsRoutes()"]
    
    posts --> postRoute["Route per post<br/>path: permalink"]
    paginated --> pageRoute["Route per page<br/>path: baseUrl/page/N"]
    tags --> tagsList["Tags list route<br/>path: blogBasePath/tags"]
    tags --> tagPages["Routes per tag page<br/>path: tag.permalink/page/N"]
    archive --> archiveRoute["Archive route<br/>path: blogBasePath/archive"]
    authors --> authorsList["Authors list route<br/>path: authorsListPath"]
    authors --> authorPages["Routes per author page<br/>path: author.page.permalink/page/N"]
```

### Blog post routes

Individual blog post routes are created via`createBlogPostRoute`:

```ts
function createBlogPostRoute(blogPost: BlogPost): RouteConfig {
  return {
    path: blogPost.metadata.permalink,
    component: blogPostComponent,
    exact: true,
    modules: {
      sidebar: sidebarModulePath,
      content: blogPost.metadata.source,
    },
    metadata: createBlogPostRouteMetadata(blogPost.metadata),
    context: {
      blogMetadata: blogMetadataModulePath,
    },
  };
}
```

Each route receives:
-`sidebar`: path to the sidebar props module (recent posts list)
-`content`: path to the MDX source file
-`metadata`: file path and last update info for the route metadata
-`context`: blog metadata module containing base paths and titles

### Paginated blog listing routes

`createBlogPostsPaginatedRoutes` uses`paginateBlogPosts` to split listed posts into pages and create a route for each:

```ts
function createBlogPostsPaginatedRoutes(): RouteConfig[] {
  const blogListPaginated = paginateBlogPosts({
    blogPosts: listedBlogPosts,
    blogTitle,
    blogDescription,
    postsPerPageOption: postsPerPage,
    basePageUrl: blogBasePath,
    page