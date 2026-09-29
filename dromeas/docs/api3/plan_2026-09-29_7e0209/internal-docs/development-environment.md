# Development environment

This guide covers setting up the Docusaurus monorepo locally for development and testing. The repository packages use version `4.0.0` and are published as a pnpm workspace.

## Prerequisites

- **Node.js 24.21 or later**. The `engines` field in `packages/docusaurus/package.json` and `packages/lqip-loader/package.json` sets `"node": ">=24.21"`.
- **pnpm**. Scripts across the workspace use `pnpm exec` and dependencies use the `workspace:*` protocol, which requires pnpm.
- **Git** for cloning the repository.

Verify your Node.js version with:

```sh
node --version
```

## Clone and install

1. Clone the repository:

   ```sh
   git clone https://github.com/Mohita111/docusaurus.git
   cd docusaurus
   ```

2. Install workspace dependencies:

   ```sh
   pnpm install
   ```

The prefetched source files do not include the root `package.json`, but `website/package.json` depends on local packages through `"workspace:*"` entries such as `"@docusaurus/core": "workspace:*"`, confirming pnpm workspace behavior.

## Run the website locally

The main development target is the Docusaurus website under `website/`.

Start the development server with:

```sh
cd website
pnpm start
```

Or from the repository root:

```sh
pnpm --dir website start
```

The underlying script in `website/package.json` is:

```json
"start": "docusaurus start"
```

## Common website commands

All commands in this section are defined in `website/package.json`.

| Command | Script | Purpose |
|---|---|---|
| `pnpm start` | `docusaurus start` | Start the local dev server |
| `pnpm build` | `docusaurus build` | Produce the static site |
| `pnpm serve` | `docusaurus serve` | Serve the built static site |
| `pnpm clear` | `docusaurus clear && rimraf changelog && rimraf _dogfooding/_swizzle_theme_tests` | Clear the Docusaurus cache and generated test content |
| `pnpm typecheck` | `tsc` | Type-check `website` TypeScript code |
| `pnpm build:fast` | `cross-env BUILD_FAST=true pnpm build --locale en` | Build only the English locale with fast mode |
| `pnpm build:fast:rsdoctor` | `cross-env BUILD_FAST=true RSDOCTOR=true pnpm build --locale en` | Build with Rsdoctor diagnostics |
| `pnpm start:baseUrl` | `cross-env BASE_URL='/build/' pnpm start` | Start the dev server with a custom base URL |
| `pnpm build:baseUrl` | `cross-env BASE_URL='/build/' pnpm build` | Build with a custom base URL |
| `pnpm start:blogOnly` | `cross-env pnpm start --config=docusaurus.config-blog-only.js` | Start using the blog-only config |
| `pnpm build:blogOnly` | `cross-env pnpm build --config=docusaurus.config-blog-only.js` | Build using the blog-only config |

## Build workspace packages

The Docusaurus core package lives in `packages/docusaurus/package.json`.

To build `@docusaurus/core` from its package directory:

```sh
cd packages/docusaurus
pnpm build
```

The build script is:

```json
"build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js"
```

To watch `@docusaurus/core` during development:

```sh
cd packages/docusaurus
pnpm watch
```

The watch script is:

```json
"watch": "run-p -c copy:watch build:watch"
```

The low-quality image placeholder loader package in `packages/lqip-loader/package.json` has a simpler build pipeline:

```sh
cd packages/lqip-loader
pnpm build
```

Its scripts are:

```json
"build": "tsc",
"watch": "tsc --watch"
```

The `@docusaurus/core` package exposes its CLI through the `bin` field:

```json
"bin": {
  "docusaurus": "bin/docusaurus.mjs"
}
```

## Visual regression tests

Visual diff tests are configured in `argos/package.json`.

| Command | Script | Purpose |
|---|---|---|
| `pnpm screenshot` | `playwright test` | Run Playwright screenshot tests |
| `pnpm upload` | `pnpm exec -- argos upload ./screenshots/chromium` | Upload Chromium screenshots to Argos |
| `pnpm upload-text-snapshots` | `pnpm exec -- argos upload "../website/build" --build-name text-snapshots --files styles.css --files "docs/**/*.html" --files "blog/**/*.html"` | Upload text snapshots from the website build |
| `pnpm report` | `playwright show-report` | Open the local Playwright report |

Run these commands from the `argos` directory:

```sh
cd argos
pnpm screenshot
```

## Netlify and Crowdin scripts

Production deployment scripts are defined in `website/package.json`.

- `pnpm netlify:build:production` runs translation generation, uploads sources to Crowdin, downloads translations, and then builds:

  ```json
  "netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build"
  ```

- `pnpm netlify:build:branchDeploy` and `pnpm netlify:build:deployPreview` both run `pnpm build`.
- `pnpm netlify:crowdin:downloadTranslations` waits for Crowdin and downloads website translations through the root workspace:

  ```json
  "netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website"
  ```

## Swizzle tests

The website includes automated tests for theme swizzling behavior. Each command sets `SWIZZLE_ACTION` and `SWIZZLE_TYPESCRIPT` before invoking `_dogfooding/testSwizzleThemeClassic.mjs`.

| Command | Script |
|---|---|
| `pnpm test:swizzle:eject:js` | `cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs` |
| `pnpm test:swizzle:eject:ts` | `cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs` |
| `pnpm test:swizzle:wrap:js` | `cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs` |
| `pnpm test:swizzle:wrap:ts` | `cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs` |

Run them from `website`:

```sh
cd website
pnpm test:swizzle:eject:js
```

## Environment variables

The following environment variables are used by scripts in `website/package.json`:

- `BASE_URL` overrides the base URL in `start:baseUrl` and `build:baseUrl`.
- `BUILD_FAST` enables fast build mode in `build:fast` and `build:fast:rsdoctor`.
- `RSDOCTOR` enables Rsdoctor diagnostics in `build:fast:rsdoctor`.
- `SWIZZLE_ACTION` selects `eject` or `wrap` in swizzle tests.
- `SWIZZLE_TYPESCRIPT` selects JavaScript or TypeScript output in swizzle tests.
- `DOCUSAURUS_EXIT_AFTER_BUNDLING` stops the build after bundling in `profile:bundle:cpu`.

The CPU profiling script in `website/package.json` is:

```json
"profile:bundle:cpu": "DOCUSAURUS_EXIT_AFTER_BUNDLING=true node --cpu-prof --cpu-prof-dir .cpu-prof ./node_modules/.bin/docusaurus build --locale en"
```

## Test fixture package

`admin/test-bad-package/package.json` is a private fixture that intentionally declares old dependency versions:

```json
"dependencies": {
  "@mdx-js/react": "1.0.1",
  "react": "16.14.0",
  "react-dom": "16.14.0"
}
```

This fixture has no scripts and is not referenced by the scripts in the provided source files.