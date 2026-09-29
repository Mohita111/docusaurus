# Blog plugin internals

This document describes the internal architecture of `@docusaurus/plugin-content-blog`, located in `packages/docusaurus-plugin-content-blog/src`. The plugin reads Markdown/MDX files, parses front matter, builds `BlogPost` objects and associated tags/authors, registers routes, and emits RSS, Atom, and JSON feeds.

The source files covered here are:

- `authors.ts`
- `authorsMap.ts` referenced but not included
- `authorsProblems.ts` referenced but not included
- `authorsSocials.ts` referenced but not included
- `blogUtils.ts`
- `contentHelpers.ts`
- `feed.ts`
- `frontMatter.ts` referenced but not included
- `index.ts`
- `markdownLoader.ts`
- `options.ts`
- `props.ts` referenced but not included
- `readingTime.ts` referenced but not included
- `routes.ts`
- `translations.ts` referenced but not included

Behavior for referenced but not included modules is described only through their call sites.

## Module overview

| File | Responsibility |
| --- | --- |
| `index.ts` | Plugin factory, lifecycle methods, webpack MDX loader registration |
| `options.ts` | Default options, Joi validation schema, `validateOptions` export |
| `blogUtils.ts` | Blog post file discovery, parsing, pagination, tags, date resolution |
| `authors.ts` | Author normalization and grouping |
| `routes.ts` | Route generation for posts, paginated lists, tags, authors, and archives |
| `markdownLoader.ts` | Webpack loader that truncates blog content |
| `feed.ts` | RSS, Atom, and JSON feed generation and HTML head tags |
| `contentHelpers.ts` | Mutable maps shared between content loading and webpack MDX processing |

## Runtime data flow

1. Docusaurus validates the blog plugin options with `validateOptions` exported from `options.ts`.

2. During `loadContent`, `index.ts` loads the authors map and tags file in parallel:

   ```ts
   const {authorsMap, tagsFile} = await combinePromises({
     authorsMap: getAuthorsMapChecked(),
     tagsFile: getTagsFile({contentPaths, tags: options.tags}),
   });
   ```

3. `generateBlogPosts` in `blogUtils.ts` finds Markdown/MDX files with `Globby`, processes each file through `processBlogSourceFile`, filters out drafts, and sorts posts by date.

4. `applyProcessBlogPosts` in `blogUtils.ts` invokes the user-provided `processBlogPosts` hook:

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

5. `loadContent` collocates `prevItem` and `nextItem` on listed blog posts and computes paginated tag data by calling `getBlogTags`.

6. `contentLoaded` calls `contentHelpers.updateContent(content)` to populate `sourceToBlogPost` and `sourceToPermalink`, then calls `createAllRoutes` from `routes.ts`.

7. The generated route configuration points post content modules at the original MDX source path. During webpack builds, the blog MDX loader rule from `index.ts` processes those files. `markdownLoader.ts` truncates the source when the resource query contains `truncated=true`.

8. `postBuild` emits feed files by calling `createBlogFeedFiles` in `feed.ts`.

9. `injectHtmlTags` adds RSS, Atom, and JSON feed link tags to the document head.

## `index.ts`

`index.ts` exports `pluginContentBlog` as the default plugin factory:

```ts
export default async function pluginContentBlog(
  context: LoadContext,
  options: PluginOptions,
): Promise<Plugin<BlogContent>>
```

### Content paths and data directory

The plugin constructs localized content paths:

```ts
const contentPaths: BlogContentPaths = {
  contentPath: path.resolve(siteDir, options.path),
  contentPathLocalized: shouldTranslate
    ? getPluginI18nPath({
        localizationDir,
        pluginName: PluginName,
        pluginId: options.id,
      })
    : undefined,
};
```

Generated data files are stored under:

```ts
const pluginDataDirRoot = path.join(generatedFilesDir, PluginName);
const dataDir = path.join(pluginDataDirRoot, pluginId);
```

The `aliasedSource` function maps generated module paths to the `~blog` webpack alias:

```ts
const aliasedSource = (source: string) =>
  `~blog/${posixPath(path.relative(pluginDataDirRoot, source))}`;
```

### MDX loader rule

`createBlogMDXLoaderRule` registers a webpack rule for Markdown/MDX content. It uses `createMDXLoaderItem` from `@docusaurus/mdx-loader` with these blog-specific callbacks:

- `ftp` uses `footnoteIDFixer` before default remark plugins.
- `metadataPath` must be in sync with the path created in `routes.ts`:

  ```ts
  metadataPath: (mdxPath: string) => {
    const aliasedPath = aliasedSitePath(mdxPath, siteDir);
    return path.join(dataDir, `${docuHash(aliasedPath)}.json`);
  }
  ```

- `createAssets` looks up the current blog post in `contentHelpers.sourceToBlogPost` and returns the front matter image and author image URLs.
- `resolveMarkdownLink` resolves Markdown links through `contentHelpers.sourceToPermalink`.

The rule includes all content directories with a trailing path separator:

```ts
include: contentDirs
  .map(addTrailingPathSeparator),
```

### Lifecycle methods

| Lifecycle method | Behavior |
| --- | --- |
| `getPathsToWatch` | Returns authors map path, tags file paths, and Markdown/MDX content globs |
| `getTranslationFiles` | Calls `getTranslationFiles` from `translations.ts` |
| `loadContent` | Loads authors/tags, generates blog posts, applies `processBlogPosts`, creates paginated tags, and returns `BlogContent` |
| `contentLoaded` | Updates content helpers and calls `createAllRoutes` |
| `translateContent` | Calls `translateContent` from `translations.ts` |
| `configureWebpack` | Adds the `~blog` alias and blog MDX loader rule |
| `postBuild` | Emits feed files when the feed type is configured |
| `injectHtmlTags` | Adds feed alternate link tags |

## `options.ts`

`DEFAULT_OPTIONS` is exported and used both as the schema base and for direct plugin defaults.

```ts
export const DEFAULT_OPTIONS: PluginOptions = {
  id: DEFAULT_PLUGIN_ID,
  feedOptions: {
    type: ['rss', 'atom'],
    copyright: '',
    limit: 20,
    xslt: {
      rss: null,
      atom: null,
    },
  },
  beforeDefaultRehypePlugins: [],
  beforeDefaultRemarkPlugins: [],
  admonitions: true,
  truncateMarker: /<!--\s*truncate\s*-->|\{\/\*\s*truncate\s*\*\/\}/,
  rehypePlugins: [],
  remarkPlugins: [],
  recmaPlugins: [],
  showReadingTime: true,
  blogTagsPostsComponent: '@theme/BlogTagsPostsPage',
  blogTagsListComponent: '@theme/BlogTagsListPage',
  blogAuthorsPostsComponent: '@theme/Blog/Pages/BlogAuthorsPostsPage',
  blogAuthorsListComponent: '@theme/Blog/Pages/BlogAuthorsListPage',
  blogPostComponent: '@theme/BlogPostPage',
  blogListComponent: '@theme/BlogListPage',
  blogArchiveComponent: '@theme/BlogArchivePage',
  blogDescription: 'Blog',
  blogTitle: 'Blog',
  blogSidebarCount: 5,
  blogSidebarTitle: 'Recent posts',
  postsPerPage: 10,
  include: ['**/*.{md,mdx}'],
  exclude: GlobExcludeDefault,
  routeBasePath: 'blog',
  tagsBasePath: 'tags',
  archiveBasePath: 'archive',
  pageBasePath: 'page',
  path: 'blog',
  editLocalizedFiles: false,
  authorsMapPath: 'authors.yml',
  readingTime: ({content, defaultReadingTime, locale}) =>
    defaultReadingTime({content, locale}),
  sortPosts: 'descending',
  showLastUpdateTime: false,
  showLastUpdateAuthor: false,
  processBlogPosts: async () => undefined,
  tags: undefined,
  authorsBasePath: 'authors',
  onInlineTags: 'warn',
  onInlineAuthors: 'warn',
  onUntruncatedBlogPosts: 'warn',
};
```

### Built-in XSLT paths

`XSLTBuiltInPaths` references asset files shipped with the package:

```ts
export const XSLTBuiltInPaths = {
  rss: path.resolve(__dirname, '..', 'assets', 'rss.xsl'),
  atom: path.resolve(__dirname, '..', 'assets', 'atom.xsl'),
};
```

The XSLT schema supports a string path, `true` to use the built-in path, `false` or `null` to disable XSLT, or an object with separate `rss` and `atom` values.

### Validation

`PluginOptionSchema` is a Joi object that validates every plugin option. The export is the validation entry point:

```ts
export function validateOptions({
  validate,
  options,
}: OptionValidationContext<Options | undefined, PluginOptions>): PluginOptions {
  const validatedOptions = validate(PluginOptionSchema, options);
  return validatedOptions;
}
```

## `blogUtils.ts`

`blogUtils.ts` contains the main content processing pipeline.

### Core blog post processing

`generateBlogPosts` discovers and processes all blog source files:

```ts
export async function generateBlogPosts(
  contentPaths: BlogContentPaths,
  context: LoadContext,
  options: PluginOptions,
  tagsFile: TagsFile | null,
  authorsMap?: AuthorsMap,
): Promise<BlogPost[]> {
  const {include, exclude} = options;

  if (!(await fs.pathExists(contentPaths.contentPath))) {
    return [];
  }

  const blogSourceFiles = await Globby(include, {
    cwd: contentPaths.contentPath,
    ignore: exclude,
  });

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

  const blogPosts = (
    await Promise.all(blogSourceFiles.map(doProcessBlogSourceFile))
  ).filter(Boolean) as BlogPost[];

  blogPosts.sort(
    (a, b) => b.metadata.date.getTime() - a.metadata.date.getTime(),
  );

  if (options.sortPosts === 'ascending') {
    return blogPosts.reverse();
  }
  return blogPosts;
}
```

`processBlogSourceFile` is the single-file pipeline:

1. Resolves the content path through `getFolderContainingFile`, preferring a localized folder.
2. Calls `parseBlogPostMarkdownFile`, which reads the file, calls `parseMarkdownFile`, removes the content title, and validates front matter with `validateBlogPostFrontMatter`.
3. Constructs the aliased source path with `aliasedSitePath`.
4. Resolves `lastUpdate` with `readLastUpdateData`.
5. Returns `undefined` for drafts.

The function then determines:

- `id` and default `slug` from the file name
- `date` from front matter, file name, VCS, or file stats
- `title` from front matter, content title, or file name text
- `description` from front matter or excerpt
- `permalink` from `baseUrl`, `routeBasePath`, and `slug`
- `editUrl` from the `editUrl` option
- `tags` via `normalizeTags`
- `authors` via `getBlogPostAuthors`
- `readingTime` when `showReadingTime` is enabled
- `hasTruncateMarker` by testing `options.truncateMarker` against the raw content

### Filename and date parsing

`parseBlogFileName` uses this regular expression to extract dates from blog file names:

```ts
const DATE_FILENAME_REGEX =
  /^(?<folder>.*)(?<date>\d{4}[-/]\d{1,2}[-/]\d{1,2})[-/]?(?<text>.*?)(?:\/index)?.mdx?$/;
```

Date handling in `parseFrontMatterDate` and `getBlogPostDate` follows this precedence:

1. Front matter `date`
2. File name date
3. VCS file creation info
4. File system birth time

String dates without an explicit timezone are treated as UTC by appending `Z`:

```ts
const hasTimeZone = /[t \t]\d.*(?:z|[+-]\d{1,2}(?::?\d{2})?)$/i.test(
  dateString,
);
return new Date(hasTimeZone ? dateString : `${dateString}Z`);
```

### Pagination

`paginateBlogPosts` creates `BlogPaginated[]`. Each page includes:

```ts
{
  items: blogPosts
    .slice(page * postsPerPage, (page + 1) * postsPerPage)
    .map((item) => item.id),
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
}
```

`postsPerPage` defaults to `10`. Setting it to `'ALL'` produces a single page.

### Tag aggregation

`getBlogTags` groups posts by their `metadata.tags` and then paginates each tag:

```ts
const groups = groupTaggedItems(
  blogPosts,
  (blogPost) => blogPost.metadata.tags,
);
```

Each returned tag includes:

- `inline`
- `label`
- `permalink`
- `description`
- `items`
- `pages`
- `unlisted`

### Untruncated post reporting

`reportUntruncatedBlogPosts` compares posts against `metadata.hasTruncateMarker` and reports warnings through `logger.report`.

## `authors.ts`

### Author normalization

`getBlogPostAuthors` handles both the legacy `author_*` front matter fields and the modern `authors` front matter field.

Legacy fields are:

```ts
const name = frontMatter.author;
const title = frontMatter.author_title ?? frontMatter.authorTitle;
const url = frontMatter.author_url ?? frontMatter.authorURL;
const imageURL = normalizeImageUrl({
  imageURL: frontMatter.author_image_url ?? frontMatter.authorImageURL,
  baseUrl,
});
```

Mixing legacy author fields with the modern `authors` field throws:

```ts
throw new Error(
  `To declare blog post authors, use the 'authors' front matter in priority.
Don't mix 'authors' with other existing 'author_*' front matter. Choose one or the other, not both at the same time.`,
);
```

### Global authors map resolution

When an author is declared by key, `getAuthorsMapAuthor` requires a non-empty authors map:

```ts
if (!authorsMap || Object.keys(authorsMap).length === 0) {
  throw new Error(`Can't reference blog post authors by a key (such as '${key}') because no authors map file could be loaded. ...`);
}
```

Global author values can be overridden locally by front matter through the spread in `toAuthor`:

```ts
const author = {
  ...getAuthorsMapAuthor(frontMatterAuthor.key),
  ...frontMatterAuthor,
} as Author;
```

### Grouping by author

`groupBlogPostsByAuthorKey` groups only posts that reference an author by key:

```ts
return _.mapValues(authorsMap, (author, key) =>
  blogPosts.filter((p) => p.metadata.authors.some((a) => a.key === key)),
);
```

## `routes.ts`

`routes.ts` exports both `createAllRoutes` and `buildAllRoutes`.

```ts
export async function createAllRoutes(
  param: CreateAllRoutesParam,
): Promise<void> {
  const routes = await buildAllRoutes(param);
  routes.forEach(param.actions.addRoute);
}
```

### Generated data modules

Before constructing routes, `buildAllRoutes` creates three data modules through `actions.createData`:

1. Sidebar module:

   ```ts
   const modulePath = await createData(