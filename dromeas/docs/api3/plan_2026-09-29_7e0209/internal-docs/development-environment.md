# Development Environment

This guide covers local development for the Docusaurus monorepo. The setup is based on the package files in `website/package.json`, `packages/docusaurus/package.json`, `packages/lqip-loader/package.json`, and `argos/package.json`.

## Prerequisites

- Node.js `>=24.21`. This is declared in `engines` in both `packages/docusaurus/package.json` and `packages/lqip-loader/package.json`.
- pnpm. The website and Argos scripts invoke `pnpm` directly. For example, `website/package.json` uses `pnpm start` and `argos/package.json` uses `pnpm exec -- argos upload`.
- Git.

The repository uses pnpm workspace dependencies. All internal Docusaurus packages in `website/package.json` are declared with `workspace:*`, for example:

```json
"@docusaurus/core": "workspace:*",
"@docusaurus/theme-classic": "workspace:*",
"@docusaurus/preset-classic": "workspace:*"
```

## Repository layout

- `website/` — the Docusaurus documentation site, used for local development and dogfooding
- `packages/docusaurus/` — the core package published as `@docusaurus/core`
- `packages/lqip-loader/` — the Low Quality Image Placeholders webpack loader
- `argos/` — Playwright-based visual regression tests
- `admin/test-bad-package/` — fixture package for dependency validation

## Install dependencies

Clone the repository and install workspace dependencies:

```bash
git clone https://github.com/facebook/docusaurus.git
cd docusaurus
pnpm install
```

The repository URL is taken from the `repository` field in `packages/docusaurus/package.json`. The root install command is not included in the provided source set, but it is required to resolve the `workspace:*` packages.

## Start the development website

Run the website locally from the `website/` directory:

```bash
cd website
pnpm start
```

This runs the `docusaurus start` script declared in `website/package.json`.

To start with `BASE_URL=/build/`:

```bash
cd website
pnpm start:baseUrl
```

This runs:

```bash
cross-env BASE_URL='/build/' pnpm start
```

To start using the blog-only configuration:

```bash
cd website
pnpm start:blogOnly
```

This runs:

```bash
cross-env pnpm start --config=docusaurus.config-blog-only.js
```

## Develop against core packages

When you change TypeScript source files in `packages/docusaurus`, run the watch script from that package:

```bash
cd packages/docusaurus
pnpm watch
```

The `watch` script in `packages/docusaurus/package.json` runs:

```bash
run-p -c copy:watch build:watch
```

`build:watch` runs `tsc --build --watch`, and `copy:watch` runs `node ../../admin/scripts/copyUntypedFiles.js --watch`.

Keep the core watch process running in one terminal while the website runs in another terminal. The website depends on `@docusaurus/core` through the workspace protocol, so local changes are picked up through the workspace resolution.

## Type-check the website

Run TypeScript checking from the website package:

```bash
cd website
pnpm typecheck
```

The script in `website/package.json` is:

```json
"typecheck": "tsc"
```

## Build the website

Create a production build:

```bash
cd website
pnpm build
```

This runs `docusaurus build`.

For a faster build restricted to the `en` locale:

```bash
cd website
pnpm build:fast
```

The underlying command is:

```bash
cross-env BUILD_FAST=true pnpm build --locale en
```

To run the fast build with Rsdoctor enabled:

```bash
cd website
pnpm build:fast:rsdoctor
```

The underlying command is:

```bash
cross-env BUILD_FAST=true RSDOCTOR=true pnpm build --locale en
```

## Serve the built website

After building, serve the production output:

```bash
cd website
pnpm serve
```

This runs `docusaurus serve`.

## Clear local caches

Clear Docusaurus caches and generated test artifacts:

```bash
cd website
pnpm clear
```

The script runs:

```bash
docusaurus clear && rimraf changelog && rimraf _dogfooding/_swizzle_theme_tests
```

## Run swizzle theme tests

The website package includes four swizzle test scripts. They vary `SWIZZLE_ACTION` and `SWIZZLE_TYPESCRIPT` and run `_dogfooding/testSwizzleThemeClassic.mjs`.

| Script | Environment |
| --- | --- |
| `pnpm test:swizzle:eject:js` | `SWIZZLE_ACTION=eject`, `SWIZZLE_TYPESCRIPT=false` |
| `pnpm test:swizzle:eject:ts` | `SWIZZLE_ACTION=eject`, `SWIZZLE_TYPESCRIPT=true` |
| `pnpm test:swizzle:wrap:js` | `SWIZZLE_ACTION=wrap`, `SWIZZLE_TYPESCRIPT=false` |
| `pnpm test:swizzle:wrap:ts` | `SWIZZLE_ACTION=wrap`, `SWIZZLE_TYPESCRIPT=true` |

Example:

```bash
cd website
pnpm test:swizzle:eject:js
```

## Write translations and heading IDs

Update translations from the website content:

```bash
cd website
pnpm write-translations
```

This runs `docusaurus write-translations`.

Write explicit heading IDs:

```bash
cd website
pnpm write-heading-ids
```

This runs `docusaurus write-heading-ids`.

## Run visual regression tests

The `argos/` package uses Playwright and the Argos CLI. Run the screenshot suite:

```bash
cd argos
pnpm screenshot
```

This runs `playwright test`.

Review the Playwright report:

```bash
cd argos
pnpm report
```

This runs `playwright show-report`.

Upload screenshots to Argos:

```bash
cd argos
pnpm upload
```

This runs:

```bash
pnpm exec -- argos upload ./screenshots/chromium
```

Upload text snapshots from the website build:

```bash
cd argos
pnpm upload-text-snapshots
```

This uploads `../website/build` using the build name `text-snapshots` and includes `styles.css`, `docs/**/*.html`, and `blog/**/*.html`.

## Build individual packages

### `@docusaurus/core`

Build the core package:

```bash
cd packages/docusaurus
pnpm build
```

The script runs:

```bash
tsc --build && node ../../admin/scripts/copyUntypedFiles.js
```

Watch mode:

```bash
cd packages/docusaurus
pnpm watch
```

### `@docusaurus/lqip-loader`

Build the LQIP loader:

```bash
cd packages/lqip-loader
pnpm build
```

The script runs:

```bash
tsc
```

Watch mode:

```bash
cd packages/lqip-loader
pnpm watch
```

## Environment variables

The following environment variables are used by the website scripts.

| Variable | Used by | Accepted values |
| --- | --- | --- |
| `BASE_URL` | `start:baseUrl`, `build:baseUrl` | URL path, for example `/build/` |
| `BUILD_FAST` | `build:fast`, `build:fast:rsdoctor` | `true` |
| `RSDOCTOR` | `build:fast:rsdoctor` | `true` |
| `SWIZZLE_ACTION` | swizzle tests | `eject` or `wrap` |
| `SWIZZLE_TYPESCRIPT` | swizzle tests | `true` or `false` |
| `DOCUSAURUS_EXIT_AFTER_BUNDLING` | `profile:bundle:cpu` | `true` |

## Required Node version

Both `packages/docusaurus/package.json` and `packages/lqip-loader/package.json` declare:

```json
"engines": {
  "node": ">=24.21"
}
```

Use Node.js 24.21 or later to run development and build commands.