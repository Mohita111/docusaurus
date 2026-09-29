# Client-Side Redirects Internals

This document describes the modules, services, data flow, APIs, and extension points behind the Docusaurus client-side redirects plugin.

## Overview

The plugin is located at:

```
packages/docusaurus-plugin-client-redirects/src/
```

It runs in the `postBuild` lifecycle and generates static HTML redirect pages. Each generated page contains a small script that redirects the browser from a `from` pathname to an existing `to` pathname.

Core modules:

| Module | Responsibility |
| --- | --- |
| `index.ts` | Plugin entry point and `postBuild` orchestration |
| `options.ts` | User option validation and defaults |
| `types.ts` | Shared TypeScript types |
| `collectRedirects.ts` | Collect, normalize, validate, and filter redirects |
| `extensionRedirects.ts` | Generate redirects from file extensions |
| `redirectValidation.ts` | Validate a single redirect item |
| `createRedirectPageContent.ts` | Render the HTML content of a redirect page |
| `writeRedirectFiles.ts` | Convert redirect items to files and write them (imported but not included in this source bundle) |

## Data flow

1. Docusaurus calls the plugin after the site build.
2. The plugin builds a `PluginContext` from route paths, base URL, output directory, options, and site config.
3. `collectRedirects` creates redirect items from:
   - `fromExtensions` and `toExtensions`
   - the `redirects` option
   - the `createRedirects` callback
4. Collected redirects are normalized, validated, and filtered.
5. `toRedirectFiles` converts redirect items into file descriptors.
6. `writeRedirectFiles` writes the HTML files to the output directory.

## Key types

### `RedirectItem`

Defined in `packages/docusaurus-plugin-client-redirects/src/types.ts`:

```ts
export type RedirectItem = {
  /** Pathname of the new page we should create */
  from: string;
  /** Pathname of an existing Docusaurus page */
  to: string;
};
```

The `from` path is the file that will be created. The `to` path is an existing Docusaurus route.

### `PluginContext`

Defined in the same file:

```ts
export type PluginContext = Pick<Props, 'outDir' | 'baseUrl' | 'siteConfig'> & {
  options: PluginOptions;
  relativeRoutesPaths: string[];
};
```

`relativeRoutesPaths` contains all existing route paths relative to `baseUrl`.

### `PluginOptions`

Defined in `options.ts`:

```ts
export type PluginOptions = {
  id: string;
  fromExtensions: string[];
  toExtensions: string[];
  redirects: RedirectOption[];
  createRedirects?: (path: string) => string[] | string | null | undefined;
};
```

`RedirectOption` is:

```ts
export type RedirectOption = {
  to: string;
  from: string | string[];
};
```

## Plugin entry point

File: `packages/docusaurus-plugin-client-redirects/src/index.ts`

The default export is the plugin factory:

```ts
export default function pluginClientRedirectsPages(
  context: LoadContext,
  options: PluginOptions,
): Plugin<void> | null
```

It reads `trailingSlash` and the configured router:

```ts
const {trailingSlash} = context.siteConfig;
const router = context.siteConfig.future.experimental_router;

if (router === 'hash') {
  logger.warn(
    `${PluginName} does not support the Hash Router and will be disabled.`,
  );
  return null;
}
```

For normal sites, it returns a plugin with a `postBuild` hook:

```ts
async postBuild(props) {
  const pluginContext: PluginContext = {
    relativeRoutesPaths: props.routesPaths.map(
      (path) => `${addLeadingSlash(removePrefix(path, props.baseUrl))}`,
    ),
    baseUrl: props.baseUrl,
    outDir: props.outDir,
    options,
    siteConfig: props.siteConfig,
  };

  const redirects: RedirectItem[] = collectRedirects(
    pluginContext,
    trailingSlash,
  );

  const redirectFiles: RedirectFile[] = toRedirectFiles(
    redirects,
    pluginContext,
    trailingSlash,
  );

  await writeRedirectFiles(redirectFiles);
}
```

The plugin also exports `validateOptions` and the option types:

```ts
export {validateOptions} from './options';
export type {PluginOptions, Options};
```

## Options validation

File: `packages/docusaurus-plugin-client-redirects/src/options.ts`

Defaults:

```ts
export const DEFAULT_OPTIONS: Partial<PluginOptions> = {
  fromExtensions: [],
  toExtensions: [],
  redirects: [],
};
```

The validation schema uses Joi:

```ts
const UserOptionsSchema = Joi.object<PluginOptions>({
  fromExtensions: Joi.array()
    .items(isString)
    .default(DEFAULT_OPTIONS.fromExtensions),
  toExtensions: Joi.array()
    .items(isString)
    .default(DEFAULT_OPTIONS.toExtensions),
  redirects: Joi.array()
    .items(RedirectPluginOptionValidation)
    .default(DEFAULT_OPTIONS.redirects),
  createRedirects: Joi.function().maxArity(1),
}).default(DEFAULT_OPTIONS);
```

Each redirect option must have a valid `from` and `to`:

```ts
const RedirectPluginOptionValidation = Joi.object<RedirectOption>({
  from: Joi.alternatives().try(
    PathnameSchema.required(),
    Joi.array().items(PathnameSchema.required()),
  ),
  to: Joi.string().required(),
});
```

The `createRedirects` callback is limited to one argument.

## Redirect collection

File: `packages/docusaurus-plugin-client-redirects/src/collectRedirects.ts`

The main function:

```ts
export default function collectRedirects(
  pluginContext: PluginContext,
  trailingSlash: boolean | undefined,
): RedirectItem[]
```

### Normalization

Before validation, every `to` value is normalized:

```ts
function normalizeRedirectTo(to: string) {
  return to.startsWith('/')
    ? applyTrailingSlash(to, {
        trailingSlash,
        baseUrl: pluginContext.baseUrl,
      })
    : to;
}
```

This allows site owners to toggle the global `trailingSlash` setting without changing every redirect option.

### Redirect sources

Redirects are created from four sources and combined into a single array:

```ts
const redirects = [
  ...createFromExtensionsRedirects(
    pluginContext.relativeRoutesPaths,
    pluginContext.options.fromExtensions,
  ),
  ...createToExtensionsRedirects(
    pluginContext.relativeRoutesPaths,
    pluginContext.options.toExtensions,
  ),
  ...createRedirectsOptionRedirects(pluginContext.options.redirects),
  ...createCreateRedirectsOptionRedirects(
    pluginContext.relativeRoutesPaths,
    pluginContext.options.createRedirects,
  ),
].map((redirect) => ({
  ...redirect,
  to: normalizeRedirectTo(redirect.to),
}));
```

### `redirects` option

```ts
function createRedirectsOptionRedirects(
  redirectsOption: PluginOptions['redirects'],
): RedirectItem[] {
  function optionToRedirects(option: RedirectOption): RedirectItem[] {
    if (typeof option.from === 'string') {
      return [{from: option.from, to: option.to}];
    }
    return option.from.map((from) => ({from, to: option.to}));
  }

  return redirectsOption.flatMap(optionToRedirects);
}
```

Each `RedirectOption` maps either a single `from` or multiple `from` paths to one `to` path.

### `createRedirects` callback

```ts
function createCreateRedirectsOptionRedirects(
  paths: string[],
  createRedirects: PluginOptions['createRedirects'],
): RedirectItem[] {
  function createPathRedirects(path: string): RedirectItem[] {
    const fromsMixed: string | string[] = createRedirects?.(path) ?? [];

    const froms: string[] =
      typeof fromsMixed === 'string' ? [fromsMixed] : fromsMixed;

    return froms.map((from) => ({from, to: path}));
  }

  return paths.flatMap(createPathRedirects);
}
```

For every existing route path, the callback receives the path and returns one path, an array of paths, `null`, or `undefined`. Each returned value becomes a `from` path that redirects to the original route.

### Validation

After collection, each redirect is validated with `validateRedirect`. Invalid redirects cause an aggregate error:

```ts
if (redirectValidationErrors.length > 0) {
  throw new Error(
    `Some created redirects are invalid:\n- ${redirectValidationErrors.join('\n- ')}\n`,
  );
}
```

The function also verifies that every pathname-like `to` value points to an existing route. It builds `allowedToPaths` from `relativeRoutesPaths`, then maps redirect `to` values that start with `/` through `URL.parse` to extract and decode the pathname.

Trailing-slash mismatches are detected with `differByTrailSlash`. If a redirect target exists only with a different trailing slash and the site has an explicit `trailingSlash` setting, the plugin reports:

```
You are trying to create client-side redirections to invalid paths.
```

It distinguishes between trailing-slash issues and paths that do not exist at all.

### Filtering duplicate and conflicting redirects

```ts
function filterUnwantedRedirects(
  redirects: RedirectItem[],
  pluginContext: PluginContext,
): RedirectItem[]
```

The function groups redirects by `from`. If multiple redirects share the same `from`, it logs a duplicate route report through:

```ts
logger.report(pluginContext.siteConfig.onDuplicateRoutes)
```

It then deduplicates with `_.uniqBy` and removes any redirect whose `from` path already exists in `relativeRoutesPaths`:

```ts
const {false: newRedirects = [], true: redirectsOverridingExistingPath = []} =
  _.groupBy(collectedRedirects, (redirect) =>
    pluginContext.relativeRoutesPaths.includes(redirect.from),
  );
```

Redirects that would override existing pages are ignored and reported.

## Extension redirects

File: `packages/docusaurus-plugin-client-redirects/src/extensionRedirects.ts`

Extension values are validated before use. An invalid extension throws an `Error` and suggests using `createRedirects`:

```ts
const ExtensionAdditionalMessage =
  'If the redirect extension system is not good enough for your use case, you can create redirects yourself with the "createRedirects" plugin option.';
```

`validateExtension` rejects empty extensions, dots, slashes, and invalid URI characters.

### `createToExtensionsRedirects`

For each existing route path ending with a configured extension, it creates a redirect from the same path without that extension:

```ts
export function createToExtensionsRedirects(
  paths: string[],
  extensions: string[],
): RedirectItem[] {
  extensions.forEach(validateExtension);

  const dottedExtensions = extensions.map(addLeadingDot);

  const createPathRedirects = (path: string): RedirectItem[] => {
    const extensionFound = dottedExtensions.find((ext) => path.endsWith(ext));
    if (extensionFound) {
      return [{from: removeSuffix(path, extensionFound), to: path}];
    }
    return [];
  };

  return paths.flatMap(createPathRedirects);
}
```

For example, `/page.html` generates a redirect from `/page` to `/page.html`.

### `createFromExtensionsRedirects`

For each existing route path that does not already end with a configured extension, it creates a redirect from a path with that extension:

```ts
export function createFromExtensionsRedirects(
  paths: string[],
  extensions: string[],
): RedirectItem[] {
  extensions.forEach(validateExtension);

  const dottedExtensions = extensions.map(addLeadingDot);

  const alreadyEndsWithAnExtension = (str: string) =>
    dottedExtensions.some((ext) => str.endsWith(ext));

  const createPathRedirects = (path: string): RedirectItem[] => {
    if (path === '' || path === '/' || alreadyEndsWithAnExtension(path)) {
      return [];
    }
    return extensions.map((ext) => ({
      from: path.endsWith('/')
        ? addTrailingSlash(`${removeTrailingSlash(path)}.${ext}`)
        : `${path}.${ext}`,
      to: path,
    }));
  };

  return paths.flatMap(createPathRedirects);
}
```

For example, `/page` generates `/page.html` if `html` is configured. The comment explains that the filename pattern intentionally handles trailing slashes as described in Docusaurus issue [#5055](https://github.com/facebook/docusaurus/issues/5055).

## Redirect page content

File: `packages/docusaurus-plugin-client-redirects/src/createRedirectPageContent.ts`

The plugin uses the Eta template engine:

```ts
import {Eta} from 'eta';
import redirectPageTemplate from './templates/redirectPage.template.html';

const eta = new Eta();
```

The compiled template is memoized:

```ts
const getCompiledRedirectPageTemplate = _.memoize(() =>
  eta.compile(redirectPageTemplate.trim()),
);
```

`renderRedirectPageTemplate` passes two values to the template:

```ts
function renderRedirectPageTemplate(data: {
  toUrl: string;
  searchAnchorForwarding: boolean;
}) {
  const compiled = getCompiledRedirectPageTemplate();
  return eta.render(compiled, data);
}
```

The `searchAnchorForwarding` decision is based on whether the target URL already contains a query string or hash:

```ts
function searchAnchorForwarding(toUrl: string): boolean {
  const url = URL.parse(toUrl, 'https://example.com');
  if (url === null) {
    return false;
  }
  const containsSearchOrAnchor = url.search || url.hash;
  return !containsSearchOrAnchor;
}
```

The exported function encodes the target URL before rendering:

```ts
export default function createRedirectPageContent({
  toUrl,
}: {
  toUrl: string;
}): string {
  return renderRedirectPageTemplate({
    toUrl: encodeURI(toUrl),
    searchAnchorForwarding: searchAnchorForwarding(toUrl),
  });
}
```

The HTML template file itself is not included in the provided source bundle.

## Redirect validation module

File: `packages/docusaurus-plugin-client-redirects/src/redirectValidation.ts`

Each redirect item is validated with a Joi schema:

```ts
const RedirectSchema = Joi.object<RedirectItem>({
  from: PathnameSchema.required(),
  to: Joi.string().required(),
});
```

`validateRedirect` uses `abortEarly: true` and `convert: false`:

```ts
export function validateRedirect(redirect: RedirectItem): void {
  const {error} = RedirectSchema.validate(redirect, {
    abortEarly: true,
    convert: false,
  });

  if (error) {
    throw new Error(
      `${JSON.stringify(redirect)} => Validation error: ${error.message}`,
    );
  }
}
```

The error includes the full redirect object so the user can identify the failing entry.

## Write redirect files API contract

The implementation file `writeRedirectFiles.ts` is not included in the provided source bundle. Based on call sites in `index.ts`, it exports:

```ts
export function toRedirectFiles(
  redirects: RedirectItem[],
  pluginContext: PluginContext,
  trailingSlash: boolean | undefined,
): RedirectFile[];

export default function writeRedirectFiles(
  redirectFiles: RedirectFile[],
): Promise<void>;

export type RedirectFile = unknown;
```

The actual `RedirectFile` shape and file-writing behavior are defined in `writeRedirectFiles.ts`, which is outside the provided context.

## Extension points

The plugin offers four main extension points through its options:

1. **`redirects` option** — Declare static redirects, each mapping one or more `from` paths to one `to` path.
2. **`createRedirects` callback** — Generate dynamic redirects for every existing route path. Return `null` or `undefined` to create no redirects for a given path.
3. **`fromExtensions`** — Create paths with an appended extension that redirect to extensionless paths.
4. **`toExtensions`** — Create extensionless paths that redirect to paths with an extension.

The global Docusaurus `trailingSlash` site configuration automatically normalizes all pathname-like `to` values.

## Error handling

- Validation errors from `validateRedirect` are aggregated and thrown as a single `Error`.
- Invalid target paths and trailing-slash mismatches are reported in a single error with actionable lists.
- Duplicate `from` redirects and redirects that would override existing pages are reported through `logger.report(siteConfig.onDuplicateRoutes)` and ignored.
- When the site uses hash routing, the plugin logs a warning and disables itself by returning `null`.

## Source references

- `packages/docusaurus-plugin-client-redirects/src/index.ts`
- `packages/docusaurus-plugin-client-redirects/src/options.ts`
- `packages/docusaurus-plugin-client-redirects/src/types.ts`
- `packages/docusaurus-plugin-client-redirects/src/collectRedirects.ts`
- `packages/docusaurus-plugin-client-redirects/src/extensionRedirects.ts`
- `packages/docusaurus-plugin-client-redirects/src/redirectValidation.ts`
- `packages/docusaurus-plugin-client-redirects/src/createRedirectPageContent.ts`