# Blog internals

This document describes the implementation of `@docusaurus/plugin-content-blog`, located in `packages/docusaurus-plugin-content-blog/src`. It covers the modules, data flow, route generation, MDX loader, feed generation, and extension points behind Docusaurus blogs.

## Architecture at a glance

The blog plugin is a single default export from `src/index.ts`:

```ts
export default async function pluginContentBlog(
  context: LoadContext,
  options: PluginOptions,
): Promise<Plugin<BlogContent>>
```

The plugin lifecycle is split across these phases:

1. **Option validation** in `src/options.ts`
2. **Content loading** in `src/index.ts` and `src/blogUtils.ts`
3. **Route generation** in `src/routes.ts`
4. **Webpack configuration** in `src/index.ts` and `src/markdownLoader.ts`
5. **Feed generation** in `src/feed.ts`
6. **Translation** in `src/translations.ts`

## Package layout

| File | Responsibility |
| --- | --- |
| `src/index.ts` | Plugin entry point, lifecycle methods, MDX loader setup |
| `src/blogUtils.ts` | Source file processing, date handling, pagination, tag aggregation |
| `src/authors.ts` | Author resolution and grouping |
| `src/routes.ts` | Route configuration generation |
| `src/feed.ts` | RSS, Atom, and JSON feed generation |
| `src/contentHelpers.ts` | Mutable maps used by the MDX loader |
| `src/markdownLoader.ts` | Webpack loader for truncating MDX content |
| `src/options.ts` | Default options and Joi validation |

## Data flow

The main data flow starts in `src/index.ts`.

### `loadContent`

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

After generating `blogPosts`, the plugin:

1. Calls `reportUntruncatedBlogPosts` to warn about missing truncation markers.
2. Filters unlisted posts with `shouldBeListed`.
3. Computes `prevItem` and `nextItem` metadata for listed posts.
4. Aggregates tags with `getBlogTags`.
5. Returns the `BlogContent` object.

### `contentLoaded`

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

`updateContent` populates the source-to-post and source-to-permalink maps before routes are built.

### `configureWebpack`

The webpack configuration:

- Registers the `~blog` alias.
- Adds an MDX loader rule for `.md` and `.mdx` files inside the blog content directories.

### `postBuild`

If the blog has posts and feed options are enabled, `postBuild` calls `createBlogFeedFiles`.

### `injectHtmlTags`

If feeds are enabled, `injectHtmlTags` exposes feed link tags through `createFeedHtmlHeadTags`.

## Blog source processing

The core source processing logic lives in `src/blogUtils.ts`.

### `generateBlogPosts`

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

1. If `contentPaths.contentPath` does not exist, return `[]`.
2. Use `Globby(include, {cwd: contentPaths.contentPath, ignore: exclude})` to find source files.
3. Process each file in parallel with `Promise.all`.
4. Filter out `undefined` values returned for draft posts.
5. Sort descending by `metadata.date`.
6. Reverse if `options.sortPosts === 'ascending'`.

```ts
blogPosts.sort(
  (a, b) => b.metadata.date.getTime() - a.metadata.date.getTime(),
);

if (options.sortPosts === 'ascending') {
  return blogPosts.reverse();
}
```

### `processBlogSourceFile`

This private function transforms a single source file into a `BlogPost`.

Steps:

1. Resolve the localized content folder with `getFolderContainingFile`.
2. Join the source directory and relative file name.
3. Parse the markdown file with `parseBlogPostMarkdownFile`.
4. Compute `aliasedSource` with `aliasedSitePath`.
5. Read last update data with `readLastUpdateData`.
6. Check draft and unlisted status with `isDraft` and `isUnlisted`.
7. If draft, return `undefined`.
8. Parse the file name date and slug with `parseBlogFileName`.
9. Resolve the final date with `getBlogPostDate`.
10. Compute title:

```ts
const title = frontMatter.title ?? contentTitle ?? parsedBlogFileName.text;
```

11. Compute slug:

```ts
const slug = frontMatter.slug ?? parsedBlogFileName.slug;
```

12. Compute permalink:

```ts
const permalink = normalizeUrl([baseUrl, routeBasePath, slug]);
```

13. Resolve tags with `normalizeTags`.
14. Resolve authors with `getBlogPostAuthors`.
15. Return:

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
    readingTime: showReadingTime ? options.readingTime(...) : undefined,
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

### `parseBlogFileName`

The date extraction regex is:

```ts
const DATE_FILENAME_REGEX =
  /^(?<folder>.*)(?<date>\d{4}[-/]\d{1,2}[-/]\d{1,2})[-/]?(?<text>.*?)(?:\/index)?.mdx?$/;
```

If the file name contains a date, the date is parsed as UTC by appending `Z`:

```ts
const date = new Date(`${dateString!}Z`);
```

The slug is built from folder and remaining text:

```ts
const slug = `/${slugDate}/${folder!}${text!}`;
```

### `getBlogPostDate`

Date resolution order:

1. Front matter `date`, normalized with `parseFrontMatterDate`
2. File name date
3. VCS file creation info
4. File system `birthtime` fallback

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

### `applyProcessBlogPosts`

This function applies the `processBlogPosts` extension point:

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

## Pagination and tags

### `paginateBlogPosts`

Signature:

```ts
export function paginateBlogPosts({
  blogPosts,
  basePageUrl,
  blogTitle,
  blogDescription,
  postsPerPageOption,
  pageBasePath,
}: ...
): BlogPaginated[]
```

Behavior:

- `postsPerPageOption === 'ALL'` uses all posts on one page.
- Page count is at least 1.
- Page 0 uses `basePageUrl`.
- Pages greater than 0 use:

```ts
normalizeUrl([basePageUrl, pageBasePath, `${page + 1}`])
```

- Each page receives `previousPage` and `nextPage` permalinks.

### `getBlogTags`

Tags are grouped with `groupTaggedItems`, filtered by visibility, paginated, and returned as `BlogTags`.

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

## Author resolution

Author logic is in `src/authors.ts`.

### `getBlogPostAuthors`

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

1. Legacy front matter fields are detected first:

```ts
const name = frontMatter.author;
const title = frontMatter.author_title ?? frontMatter.authorTitle;
const url = frontMatter.author_url ?? frontMatter.authorURL;
const imageURL = normalizeImageUrl({
  imageURL: frontMatter.author_image_url ?? frontMatter.authorImageURL,
  baseUrl,
});
```

2. If legacy author fields are present and `frontMatter.authors` is also present, the plugin throws:

```ts
throw new Error(
  `To declare blog post authors, use the 'authors' front matter in priority.
Don't mix 'authors' with other existing 'author_*' front matter. Choose one or the other, not both at the same time.`,
);
```

3. If no legacy fields exist, `getFrontMatterAuthors` resolves authors from `frontMatter.authors`.

4. String authors are treated as keys:

```ts
if (typeof authorInput === 'string') {
  return {key: authorInput};
}
```

5. Object authors are merged with the global author map entry:

```ts
const author = {
  ...getAuthorsMapAuthor(frontMatterAuthor.key),
  ...frontMatterAuthor,
} as Author;
```

6. Missing authors map entries cause errors with a list of valid keys.

### `groupBlogPostsByAuthorKey`

Groups blog posts by global author key:

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

Blog posts that only use inline authors are ignored.

## Content helpers for MDX loading

`src/contentHelpers.ts` provides mutable maps used by the MDX loader.

```ts
export function createContentHelpers() {
  const sourceToBlogPost = new Map<string, BlogPost>();
  const sourceToPermalink = new Map<string, string>();

  function updateContent(content: BlogContent): void {
    sourceToBlogPost.clear();
    sourceToPermalink.clear();
    indexBlogPostsBySource(content).forEach((value, key) => {
      sourceToBlogPost.set(key, value);
      sourceToPermalink.set(key, value.metadata.permalink);
    });
  }

  return {updateContent, sourceToBlogPost, sourceToPermalink};
}
```

The map key is the aliased source path:

```ts
aliasedSitePath(blogSourceAbsolute, siteDir)
```

These maps are consumed by the MDX loader for:

- Asset creation with `createAssets`
- Markdown link resolution with `resolveMarkdownLink`

## Route generation

Route generation is in `src/routes.ts`.

### `createAllRoutes`

```ts
export async function createAllRoutes(
  param: CreateAllRoutesParam,
): Promise<void> {
  const routes = await buildAllRoutes(param);
  routes.forEach(param.actions.addRoute);
}
```

### Route order

```ts
return [
  ...createBlogPostRoutes(),
  ...createBlogPostsPaginatedRoutes(),
  ...createTagsRoutes(),
  ...createArchiveRoute(),
  ...createAuthorsRoutes(),
];
```

### Blog post routes

Each blog post gets a route:

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

Post metadata is persisted with `createData`:

```ts
await createData(
  `${docuHash(metadata.source)}.json`,
  metadata,
);
```

This path must stay in sync with `metadataPath` in the MDX loader.

### Paginated blog list routes

```ts
function createBlogPostsPaginatedRoutes(): RouteConfig[] {
  const blogListPaginated = paginateBlogPosts({
    blogPosts: listedBlogPosts,
    blogTitle,
    blogDescription,
    postsPerPageOption: postsPerPage,
    basePageUrl: blogBasePath,
    pageBasePath,
  });

  return blogListPaginated.map((paginated) => {
    return {
      path: paginated.metadata.permalink,
      component: blogListComponent,
      exact: true,
      modules: {
        sidebar: sidebarModulePath,
        items: blogPostItemsModule(paginated.items),
      },
      props: {
        metadata: paginated.metadata,
      },
    };
  });
}
```

### Tag routes

If `blogTags` is empty, no tag routes are created.

Otherwise the plugin creates:

- One tags list route at `blogTagsListPath`
- Paginated routes for each tag

```ts
const tagsListRoute: RouteConfig = {
  path: blogTagsListPath,
  component: blogTagsListComponent,
  exact: true,
  modules: {
    sidebar: sidebarModulePath,
  },
  props: {
    tags: toTagsProp({blogTags}),
  },
};
```

### Author routes

Author routes are created only when `authorsMap` exists and is not empty.

Routes include:

- One author list route at `authorsListPath`
- Paginated routes for each author that has `author.page` set

The author list path is computed as:

```ts
const authorsListPath = normalizeUrl([blogBasePath, authorsBasePath]);
```

### Archive route

The archive route is created only when:

```ts
if (archiveBasePath && listedBlogPosts.length) { ... }
```

## MDX loader

The MDX loader rule is created in `src/index.ts` with `createBlogMDXLoaderRule`.

```ts
function createBlogMDXLoaderRule(): RuleSetRule {
  // ...
  return {
    test: /\.mdx?$/i,
    include: contentDirs.map(addTrailingPathSeparator),
    use: [mdxLoaderItem, createBlogMarkdownLoader()],
  };
}
```

The loader order is `[mdxLoaderItem, createBlogMarkdownLoader()]`.

### `metadataPath`

```ts
metadataPath: (mdxPath: string) => {
  const aliasedPath = aliasedSitePath(mdxPath, siteDir);
  return path.join(dataDir, `${docuHash(aliasedPath)}.json`);
},
```

### `createAssets`

The MDX loader resolves blog post assets through `contentHelpers.sourceToBlogPost`:

```ts
createAssets: ({filePath}: {filePath: string}): Assets => {
  const blogPost = contentHelpers.sourceToBlogPost.get(
    aliasedSitePath(filePath, siteDir),
  )!;
  if (!blogPost) {
    throw new Error(`Blog post not found for  filePath=${filePath}`);
  }
  return {
    image: blogPost.metadata.frontMatter.image as string,
    authorsImageUrls: blogPost.metadata.authors.map(
      (author) => author.imageURL,
    ),
  };
},
```

### `footnoteIDFixer`

Blog-specific remark processing prepends `footnoteIDFixer`:

```ts
beforeDefaultRemarkPlugins: [
  footnoteIDFixer,
  ...beforeDefaultRemarkPlugins,
],
```

### `markdownLoader.ts`

The custom webpack loader truncates blog post content when the resource query includes `truncated=true`.

```ts
const truncated: boolean | undefined = this.resourceQuery
  ? !!new URLSearchParams(this.resourceQuery.slice(1)).get('truncated')
  : undefined;

if (truncated) {
  finalContent = truncate(finalContent, markdownLoaderOptions.truncateMarker);
}
```

The route modules request truncated content through the query:

```ts
content: {
  __import: true,
  path: getBlogPostById(id).metadata.source,
  query: {
    truncated: true,
  },
},
```

## Feed generation

Feed logic is in `src/feed.ts`.

### Supported feed types

```ts
const FeedConfigs: Record<
  FeedType,
  {
    outputFileName: string;
    getContent: (feed: Feed) => string;
    getXsltFilePath: (xslt: FeedXSLTOptions) => string | null;
  }
> = {
  rss: {
    outputFileName: 'rss.xml',
    getContent: (feed) => feed.rss2(),
    getXsltFilePath: (xslt) => xslt.rss,
  },
  atom: {
    outputFileName: 'atom.xml',
    getContent: (feed) => feed.atom1(),
    getXsltFilePath: (xslt) => xslt.atom,
  },
  json: {
    outputFileName: 'feed.json',
    getContent: (feed) => feed.json1(),
    getXsltFilePath: () => null,
  },
};
```

### `createBlogFeedFiles`

```ts
export async function createBlogFeedFiles({
  blogPosts: allBlogPosts,
  options,
  siteConfig,
  outDir,
  locale,
  contentPaths,
}: ...
): Promise<void>
```

Steps:

1. Remove draft and unlisted posts:

```ts
const blogPosts = allBlogPosts.filter(shouldBeInFeed);
```

2. Generate the `Feed` object.
3. If no feed or no `feedOptions.type`, return.
4. Write each configured feed type.

### `generateBlogFeed`

Returns `null` when there are no blog posts.

Uses `feedOptions.limit` unless it is `false` or `null`:

```ts
const blogPostsForFeed =
  feedOptions.limit === false || feedOptions.limit === null
    ? blogPosts
    : blogPosts.slice(0, feedOptions.limit);
```

Creates the `Feed` instance with:

```ts
const feed = new Feed({
  id: blogBaseUrl,
  title: feedOptions.title ?? `${title} Blog`,
  updated,
  language: feedOptions.language ?? locale,
  link: blogBaseUrl,
  description: feedOptions.description ?? `${siteConfig.title} Blog`,
  favicon: favicon ? normalizeUrl([siteUrl, baseUrl, favicon]) : undefined,
  copyright: feedOptions.copyright,
});
```

### `defaultCreateFeedItems`

The default item generator:

1. Reads the rendered HTML output with `readOutputHTMLFile`:

```ts
const content = await readOutputHTMLFile(
  permalink.replace(baseUrl, ''),
  outDir,
  trailingSlash,
);
```

2. Loads the HTML into Cheerio.
3. Rewrites links and image URLs in the blog post container to absolute URLs.
4. Returns:

```ts
const feedItem: BlogFeedItem = {
  title: metadataTitle,
  id: blogPostAbsoluteUrl,
  link: blogPostAbsoluteUrl,
  date,
  description,
  category: tags.map((tag) => ({name: tag.label, term: tag.label})),
  content: $(`#${blogPostContainerID}`).html()!,
};
```

5. Adds authors when the array is not empty:

```ts
const feedItemAuthors = authors.map(toFeedAuthor);
if (feedItemAuthors.length > 0) {
  feedItem.author = feedItemAuthors;
}
```

### XSLT handling

RSS and Atom feeds support XSLT style sheets.

- `resolveXsltFilePaths` validates that both the XSLT file and a co-located CSS file exist.
- `generateXsltFiles` copies both files into the feed output directory.
- `injectXslt` inserts an XML style sheet declaration with a string replace:

```ts
return feedContent.replace(