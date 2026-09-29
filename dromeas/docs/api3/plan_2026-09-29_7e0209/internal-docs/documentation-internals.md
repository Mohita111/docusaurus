# Documentation Internals

This document describes the internal architecture of `@docusaurus/plugin-content-docs`, the Docusaurus plugin responsible for loading Markdown documentation, computing slugs and metadata, building sidebar structures, and generating React routes and props.

## Source module map

All paths are relative to `packages/docusaurus-plugin-content-docs/src`.

| Module | Size | Purpose |
|---|---|---|
| `index.ts` | 8,868 B | Plugin entry point and lifecycle orchestration |
| `docs.ts` | 11,651 B | Doc loading, ID indexing, category index detection |
| `props.ts` | 5,821 B | Server-rendered props transformations |
| `slug.ts` | 2,305 B | Slug computation and validation |
| `frontMatter.ts` | 2,039 B | Front matter schema and normalization |
| `numberPrefix.ts` | 2,041 B | Default number-prefix parser |
| `categoryGeneratedIndex.ts` | 1,759 B | Category generated index handling |
| `contentHelpers.ts` | 1,252 B | Content path helpers |
| `globalData.ts` | 2,106 B | Global data assembly |
| `routes.ts` | 7,279 B | React route generation |
| `translations.ts` | 11,123 B | Translation extraction |
| `cli.ts` | 5,889 B | CLI commands (docs versioning) |
| `options.ts` | 7,122 B | Options schema and validation |
| `types.ts` | 890 B | Internal types such as `VersionTag`, `VersionTags` |
| `server-export.ts` | 710 B | Node.js-only export surface for third-party plugins |
| `plugin-content-docs.d.ts` | 21,863 B | Public type declarations and theme module augmentation |
| `client/docsSidebar.tsx` | — | React context provider and `useDocsSidebar` hook |
| `client/docsSearch.ts` | — | Contextual search tag hook |
| `sidebars/` | — | Sidebar subsystem (see [Sidebar subsystem](#sidebar-subsystem)) |
| `versions/` | — | Version reading, filtering, and banners |

## Server-side APIs

### Props transformation layer (`props.ts`)

`props.ts` converts internal `LoadedVersion` data into the props consumed by React components. It imports `createDocsByIdIndex` from `./docs` and sidebar types from `./sidebars/types`.

#### `toSidebarDocItemLinkProp`

```ts
export function toSidebarDocItemLinkProp({
  item,
  doc,
}: {
  item: SidebarItemDoc;
  doc: Pick<
    DocMetadata,
    'id' | 'title' | 'permalink' | 'unlisted' | 'frontMatter'
  >;
}): PropSidebarItemLink
```

Returns a `PropSidebarItemLink` with `type: 'link'`. Front matter values take precedence over `sidebars.json` values for `label`, `className`, and `customProps`:

```ts
label: frontMatter.sidebar_label ?? item.label ?? title,
className: frontMatter.sidebar_class_name ?? item.className,
customProps: frontMatter.sidebar_custom_props ?? item.customProps,
```

The `key` is copied from `item.key` only when present.

#### `toSidebarsProp`

```ts
export function toSidebarsProp(
  loadedVersion: Pick<LoadedVersion, 'docs' | 'sidebars'>,
): PropSidebars
```

This function normalizes sidebars so every item becomes either `link` or `category`. It does the following:

1. Builds a `docsById` index with `createDocsByIdIndex(loadedVersion.docs)`.
2. Throws a descriptive error listing available doc IDs when a sidebar references a missing doc ID.
3. Resolves `doc` and `ref` items to `PropSidebarItemLink` through `convertDocLink`.
4. Resolves category link types:
   - `doc` — returns the referenced doc's `permalink`
   - `generated-index` — returns `link.permalink`
   - otherwise — `undefined`
5. Propagates `unlisted` and `frontMatter.sidebar_custom_props` from category doc links.
6. Recursively maps every item with `normalizeItem`.

The final transformation uses `_.mapValues(loadedVersion.sidebars, ...)`.

#### `toVersionMetadataProp`

```ts
export function toVersionMetadataProp(
  pluginId: string,
  loadedVersion: LoadedVersion,
): PropVersionMetadata
```

Builds the `PropVersionMetadata` value consumed by `DocVersionRoot`. It includes `docsSidebars` from `toSidebarsProp(loadedVersion)` and `docs` from `toVersionDocsProp(loadedVersion)`.

#### `toVersionDocsProp`

```ts
function toVersionDocsProp(loadedVersion: LoadedVersion): PropVersionDocs
```

Returns an object keyed by doc ID. Each value is a `PropVersionDoc` subset:

```ts
{id, title, description, sidebar}
```

#### `toTagDocListProp`

```ts
export function toTagDocListProp({
  allTagsPath,
  tag,
  docs,
}: {
  allTagsPath: string;
  tag: VersionTag;
  docs: DocMetadata[];
}): PropTagDocList
```

Converts a `VersionTag` into `PropTagDocList`. It maps `tag.docIds` to matching docs, sorts them alphabetically by title, and projects each doc to:

```ts
{id, title, description, permalink}
```

The result also includes `label`, `permalink`, `description`, `allTagsPath`, `count`, and `unlisted`.

#### `toTagsListTagsProp`

```ts
export function toTagsListTagsProp(
  versionTags: VersionTags,
): PropTagsListPage['tags']
```

Filters out unlisted tags and maps each remaining tag to `{label, permalink, description, count}`.

### Slug computation (`slug.ts`)

```ts
export default function getSlug({
  baseID,
  frontMatterSlug,
  source,
  sourceDirName,
  stripDirNumberPrefixes = true,
  numberPrefixParser = DefaultNumberPrefixParser,
}: {
  baseID: string;
  frontMatterSlug?: string;
  source: DocMetadataBase['source'];
  sourceDirName: DocMetadataBase['sourceDirName'];
  stripDirNumberPrefixes?: boolean;
  numberPrefixParser?: NumberPrefixParser;
}): string
```

Slug resolution follows this order:

1. If `frontMatterSlug` starts with `/`, it is returned unchanged.
2. Otherwise the directory slug is computed from `sourceDirName`:
   - `stripDirNumberPrefixes` controls whether `stripPathNumberPrefixes` is applied.
   - A `sourceDirName` of `'.'` resolves to `'/'`.
   - Other directories get leading and trailing slashes via `addLeadingSlash(addTrailingSlash(...))`.
3. If there is no front matter slug and the source is a category index (determined by `isCategoryIndex(toCategoryIndexMatcherParam(...))`), the directory slug is returned directly.
4. Otherwise the base slug is `frontMatterSlug ?? baseID`, resolved against the directory slug with `resolvePathname`.

The final slug is validated with `isValidPathname`. An invalid slug throws an error that suggests using front matter to set a custom slug.

Dependencies imported by `slug.ts`:

- `isValidPathname`, `resolvePathname` from `@docusaurus/utils`
- `addLeadingSlash`, `addTrailingSlash` from `@docusaurus/utils-common`
- `DefaultNumberPrefixParser`, `stripPathNumberPrefixes` from `./numberPrefix`
- `isCategoryIndex`, `toCategoryIndexMatcherParam` from `./docs`

### Node.js escape hatch (`server-export.ts`)

This module is intentionally undocumented in public docs but is used by third-party plugins. It avoids breaking changes for those consumers.

```ts
export {
  CURRENT_VERSION_NAME,
  VERSIONED_DOCS_DIR,
  VERSIONED_SIDEBARS_DIR,
  VERSIONS_JSON_FILE,
} from './constants';

export {
  filterVersions,
  getDefaultVersionBanner,
  getVersionBadge,
  getVersionBanner,
} from './versions/version';

export {readVersionNames} from './versions/files';
```

The verified exported symbols are:

- Constants: `CURRENT_VERSION_NAME`, `VERSIONED_DOCS_DIR`, `VERSIONED_SIDEBARS_DIR`, `VERSIONS_JSON_FILE`
- Version helpers: `filterVersions`, `getDefaultVersionBanner`, `getVersionBadge`, `getVersionBanner`
- Version file reading: `readVersionNames`

## Client APIs

### Sidebar context (`client/docsSidebar.tsx`)

```tsx
type ContextValue = {name: string; items: PropSidebar};

const EmptyContext: unique symbol = Symbol('EmptyContext');

const Context = React.createContext<ContextValue | null | typeof EmptyContext>(
  EmptyContext,
);
```

The context uses a unique symbol as the initial value. This distinguishes two states:

- `EmptyContext` — no `DocsSidebarProvider` is mounted, causing `useDocsSidebar` to throw
- `null` — a provider is mounted, but the current doc has no sidebar

#### `DocsSidebarProvider`

```tsx
export function DocsSidebarProvider({
  children,
  name,
  items,
}: {
  children: ReactNode;
  name: string | undefined;
  items: PropSidebar | undefined;
}): ReactNode
```

Memos the context value. The value is `null` unless both `name` and `items` are defined.

#### `useDocsSidebar`

```tsx
export function useDocsSidebar(): ContextValue | null
```

Returns the current sidebar context. Throws `ReactContextError('DocsSidebarProvider')` when called outside a provider.

### Contextual search tags (`client/docsSearch.ts`)

```ts
export function getDocsVersionSearchTag(
  pluginId: string,
  versionName: string,
): string {
  return `docs-${pluginId}-${versionName}`;
}
```

Search tags use the format `docs-<pluginId>-<versionName>`.

```ts
export function useDocsContextualSearchTags(): string[]
```

The hook computes one tag per docs plugin instance. For each plugin ID, it selects a version using this priority:

1. Active version — when `activePluginAndVersion.activePlugin.pluginId` matches the plugin ID
2. Preferred version — from `useDocsPreferredVersionByPluginId`
3. Latest version — the version where `isLast` is true, from `useAllDocsData`

This logic is deliberately generic and not coupled to Algolia/DocSearch.

## Core type system (`plugin-content-docs.d.ts`)

This ambient declaration file exposes the full public type surface and augments theme module types.

### Plugin options

The complete `PluginOptions` type is an intersection of `MetadataOptions`, `PathOptions`, `VersionsOptions`, `MDXOptions`, and `SidebarOptions`, plus plugin-specific fields:

```ts
export type PluginOptions = MetadataOptions &
  PathOptions &
  VersionsOptions &
  MDXOptions &
  SidebarOptions & {
    id: string;
    include: string[];
    exclude: string[];
    docsRootComponent: string;
    docVersionRootComponent: string;
    docRootComponent: string;
    docItemComponent: string;
    docTagDocListComponent: string;
    docTagsListComponent: string;
    docCategoryGeneratedIndexComponent: string;
    sidebarItemsGenerator: import('./sidebars/types').SidebarItemsGeneratorOption;
    tagsBasePath: string;
  };
```

`MetadataOptions` includes:

- `routeBasePath: string`
- `editUrl?: string | EditUrlFunction`
- `editCurrentVersion: boolean`
- `editLocalizedFiles: boolean`
- `showLastUpdateTime: boolean`
- `showLastUpdateAuthor: boolean`
- `numberPrefixParser: NumberPrefixParser`
- `breadcrumbs: boolean`
- plus inherited `TagsPluginOptions`

`PathOptions` contains:

- `path: string`
- `sidebarPath?: string | false | undefined`

`VersionsOptions` contains:

- `lastVersion?: string`
- `onlyIncludeVersions?: string[]`
- `disableVersioning: boolean`
- `includeCurrentVersion: boolean`
- `versions: {[versionName: string]: VersionOptions}`

`VersionOptions` per version:

- `path?: string`
- `label?: string`
- `banner?: 'none' | VersionBanner`
- `badge?: boolean`
- `noIndex?: boolean`
- `className?: string`

### Extension callback types

```ts
export type NumberPrefixParser = (filename: string) => {
  filename: string;
  numberPrefix?: number;
};
```

```ts
export type CategoryIndexMatcher = (param: {
  fileName: string;
  directories: string[];
  extension: string;
}) => boolean;
```

```ts
export type EditUrlFunction = (editUrlParams: {
  version: string;
  versionDocsDirPath: string;
  docPath: string;
  permalink: string;
  locale: string;
}) => string | undefined;
```

### Metadata types

`DocMetadataBase` carries the core doc identity:

- `id: string`
- `version: string`
- `title: string`
- `description: string`
- `source: string`
- `sourceDirName: string`
- `slug: string`
- `permalink: string`
- `draft: boolean`
- `unlisted: boolean`
- `sidebarPosition?: number`
- `editUrl?: string | null`
- `tags: TagMetadata[]`
- `frontMatter: DocFrontMatter & {[key: string]: unknown}`
- plus `LastUpdateData`

`DocMetadata` extends `DocMetadataBase` with `PropNavigation` and an optional `sidebar?: string`.

`VersionMetadata` includes:

- `versionName: string`
- `label: string`
- `path: string`
- `tagsPath: string`
- `editUrl?: string | undefined`
- `editUrlLocalized?: string | undefined`
- `banner: VersionBanner | null`
- `badge: boolean`
- `noIndex: boolean`
- `className: string`
- `isLast: boolean`
- `sidebarFilePath: string | false | undefined`
- `routePriority: number | undefined`
- plus `ContentPaths`

`LoadedVersion` combines `VersionMetadata` with `{docs, drafts, sidebars}`. `LoadedContent` is `{loadedVersions: LoadedVersion[]}`.

### Main plugin function

```ts
export default function pluginContentDocs(
  context: LoadContext,
  options: PluginOptions,
): Promise<Plugin<LoadedContent>>;

export function validateOptions(
  args: OptionValidationContext<Options | undefined, PluginOptions>,
): PluginOptions;
```

### Theme component contracts

The declaration file augments `@theme/*` modules with strict prop contracts:

| Theme module | Required props |
|---|---|
| `@theme/DocItem` | `route: DocumentRoute`, `content: PropDocContent` |
| `@theme/DocCategoryGeneratedIndexPage` | `categoryGeneratedIndex: PropCategoryGeneratedIndex` |
| `@theme/DocTagsListPage` | `tags: TagsListItem[]` |
| `@theme/DocTagDocListPage` | `tag: PropTagDocList` |
| `@theme/DocBreadcrumbs` | none |
| `@theme/DocsRoot` | `Required<RouteConfigComponentProps, 'route'>` |
| `@theme/DocVersionRoot` | route props plus `version: PropVersionMetadata` |
| `@theme/DocRoot` | `Required<RouteConfigComponentProps, 'route'>` |

`DocumentRoute` is defined inside the `@theme/DocItem` augmentation:

```ts
export type DocumentRoute = {
  readonly component: () => ReactNode;
  readonly exact: boolean;
  readonly path: string;
  readonly sidebar?: string;
};
```

## Sidebar subsystem

The `sidebars/` directory contains these modules (directory listing verified, internal contents not included in the prefetched context):

| Module | Size | Inferred role |
|---|---|---|
| `generator.ts` | 10,674 B | Autogenerated sidebar item generation |
| `utils.ts` | 15,041 B | Shared sidebar utilities |
| `types.ts` | 8,614 B | Sidebar type definitions |
| `validation.ts` | 5,732 B | Sidebar config validation |
| `index.ts` | 4,092 B | Sidebar subsystem entry point |
| `processor.ts` | 3,632 B | Sidebar processing |
| `postProcessor.ts` | 3,455 B | Post-processing transforms |
| `normalization.ts` | 2,598 B | Sidebar shape normalization |
| `README.md` | 1,243 B | Sidebar subsystem documentation |

The following type relations are verified through imports:

- `SidebarItemDoc`, `SidebarItem`, `SidebarItemCategory`, and `SidebarItemCategoryLink` are exported from `./sidebars/types` and consumed by `props.ts`.
- `SidebarItemsGeneratorOption` and `SidebarsConfig` are exported from `./sidebars/types` and referenced by `plugin-content-docs.d.ts`.
- `PropSidebarItemLink`, `PropSidebarItemHtml`, `PropSidebarItemCategory`, `PropSidebarItem`, `PropSidebarBreadcrumbsItem`, `PropSidebar`, and `PropSidebars` are re-exported from `@docusaurus/plugin-content-docs` based on `./sidebars/types`.

`props.ts` implements the normalization contract described by these types: internal items of type `doc` and `ref` are both converted to `PropSidebarItemLink` items with `type: 'link'`, and categories are recursively normalized.

## Data flow

The plugin follows a load-transform-props-render pipeline:

1. **Load** — docs are read from `options.path` and loaded as `DocMetadata[]`; sidebars are read or generated. `LoadedVersion` aggregates `VersionMetadata`, `docs`, `drafts`, and `sidebars`.
2. **Index** — `createDocsByIdIndex` builds a doc-ID → `DocMetadata` map used to resolve sidebar references.
3. **Transform to props** — `toVersionMetadataProp` converts each `LoadedVersion` into `PropVersionMetadata`. This calls:
   - `toSidebarsProp` for normalized sidebar props
   - `toVersionDocsProp` for the doc subset keyed by ID
   - `toTagsListTagsProp` / `toTagDocListProp` for tag props
4. **Route generation** — `routes.ts` (not fully inspected here) consumes the loaded content and prop types to build React Router routes wired to the theme components from `plugin-content-docs.d.ts`.
5. **Client render** — theme components such as `@theme/DocVersionRoot` receive `PropVersionMetadata`; `DocsSidebarProvider` and `useDocsSidebar` provide sidebar access to nested components; `useDocsContextualSearchTags` computes search tags for contextual search.

## Extension points

### Custom number-prefix parsing

Implement `NumberPrefixParser` and pass it through `options.numberPrefixParser`. The default implementation is `DefaultNumberPrefixParser` from `./numberPrefix`. Set to `false` to disable prefix stripping.

### Category index detection

Implement `CategoryIndexMatcher` to customize which files are treated as category index pages. The slug logic consults `isCategoryIndex(toCategoryIndexMatcherParam(...))` from `./docs`.

### Custom edit URL computation

Provide `editUrl` as an `EditUrlFunction` to control edit links per doc with access to `version`, `versionDocsDirPath`, `docPath`, `permalink`, and `locale`.

### Sidebar item generation

Provide `sidebarItemsGenerator` with the `SidebarItemsGeneratorOption` type from `./sidebars/types` to replace the default autogenerated sidebar generation.

### Theme component swizzling

All `@theme/*` contracts in `plugin-content-docs.d.ts` are extension points. Swizzle any of `DocsRoot`, `DocVersionRoot`, `DocRoot`, `DocItem`, `DocBreadcrumbs`, `DocCategoryGeneratedIndexPage`, `DocTagsListPage`, or `DocTagDocListPage` to override rendering while keeping the typed props contract.

### Third-party plugin hooks

`server-export.ts` exposes version constants, banner helpers, version filtering, and `readVersionNames` for Node.js consumers. This surface is kept stable to avoid breaking third-party plugins even though it is not part of the official documented API.

## Limitations of this document

The prefetched source context includes full content for `props.ts`, `slug.ts`, `server-export.ts`, `client/docsSidebar.tsx`, `client/docsSearch.ts`, and `plugin-content-docs.d.ts`. The following files are known only from the directory listing — their internal implementation details are not documented here:

- `index.ts`, `docs.ts`, `frontMatter.ts`, `numberPrefix.ts`, `categoryGeneratedIndex.ts`, `contentHelpers.ts`, `globalData.ts`, `routes.ts`, `translations.ts`, `cli.ts`, `options.ts`, `types.ts`, `constants.ts`
- All files under `sidebars/`
- All files under `versions/`

Inferred exports from these files are limited to symbols verified through import statements in the fully provided source files.