# CI/CD Pipeline

Docusaurus uses a multi-stage CI/CD pipeline that spans monorepo package builds, visual regression testing with Argos, translation synchronization through Crowdin, and deployments to Netlify. This document describes each stage of the pipeline and the scripts that drive it.

## Pipeline overview

The repository is a pnpm workspace with packages under `packages/` and an integration website in `website/`. The CI/CD pipeline consists of these phases:

1. **Package builds** – Compile TypeScript packages with `tsc --build`.
2. **Website builds** – Generate the static site with `docusaurus build`.
3. **Visual regression tests** – Capture screenshots with Playwright and upload to Argos.
4. **Translation sync** – Upload source strings to Crowdin and download translated content.
5. **Deployment** – Ship production and preview builds through Netlify.

Node.js 24.21 or newer is required for all packages (`"engines": { "node": ">=24.21" }`).

## Phase 1: package builds

Each package in the monorepo compiles independently with TypeScript.

The core package `@docusaurus/core` (`packages/docusaurus/package.json`) defines:

```json
"scripts": {
  "build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js",
  "watch": "run-p -c copy:watch build:watch",
  "build:watch": "tsc --build --watch",
  "copy:watch": "node ../../admin/scripts/copyUntypedFiles.js --watch"
}
```

The `copyUntypedFiles.js` admin script copies non-TypeScript assets (such as templates and default files) after the TypeScript compilation completes. During local development, `pnpm watch` runs both the TypeScript compiler and the untyped-file copier in parallel.

The LQIP loader package (`packages/lqip-loader/package.json`) follows a simpler pattern:

```json
"scripts": {
  "build": "tsc",
  "watch": "tsc --watch"
}
```

## Phase 2: website builds

The `website` package (`website/package.json`) is the integration and documentation site. Its primary build command is:

```json
"build": "docusaurus build"
```

Several build variants exist for specific needs:

| Script | Purpose |
| --- | --- |
| `build:fast` | Build only the `en` locale with fast rebuild enabled |
| `build:fast:rsdoctor` | Fast build with Rsdoctor bundle analysis enabled |
| `build:baseUrl` | Build with `BASE_URL=/build/` for hosting under a subpath |
| `build:blogOnly` | Build using the blog-only config file |

The fast build scripts set `BUILD_FAST=true` through `cross-env`:

```json
"build:fast": "cross-env BUILD_FAST=true pnpm build --locale en",
"build:fast:rsdoctor": "cross-env BUILD_FAST=true RSDOCTOR=true pnpm build --locale en"
```

## Phase 3: visual regression testing

Visual diff tests live in the `argos` package. Argos integrates with Playwright to capture screenshots and compare them against a baseline, detecting unintended visual regressions.

The scripts in `argos/package.json` are:

```json
"screenshot": "playwright test",
"upload": "pnpm exec -- argos upload ./screenshots/chromium",
"upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\"",
"report": "playwright show-report"
```

The workflow for visual testing is:

1. Run `pnpm screenshot` in the `argos` package to capture Chromium screenshots with Playwright.
2. Run `pnpm upload` in the `argos` package to publish the captured screenshots to Argos for comparison.
3. Inspect any failures with `pnpm report`, which opens the Playwright report viewer.

The `upload-text-snapshots` script uploads built website HTML files and stylesheets to Argos under a separate build named `text-snapshots`. This monitors text-level changes in the documentation site by uploading:

- `styles.css`
- Rendered HTML files from `docs/**/*.html`
- Rendered HTML files from `blog/**/*.html`

The website also has a CSS ordering assertion script:

```json
"test:css-order": "node testCSSOrder.mjs"
```

## Phase 4: translation synchronization

The website pipeline integrates Crowdin for translation management. Source strings are uploaded from the Docusaurus translation files, and translated content is downloaded before a production build.

The Crowdin-related scripts in `website/package.json` are:

```json
"netlify:crowdin:delay": "node delayCrowdin.mjs",
"netlify:crowdin:wait": "node waitForCrowdin.mjs",
"netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website",
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')",
"netlify:crowdin:uploadSources": "pnpm --dir .. crowdin:upload:website"
```

The translation pipeline has these steps:

1. **Delay** – `node delayCrowdin.mjs` inserts a delay before any Crowdin operation begins.
2. **Upload** – `pnpm --dir .. crowdin:upload:website` sends the English source strings to Crowdin.
3. **Wait** – `node waitForCrowdin.mjs` polls Crowdin until the translation service is ready.
4. **Download** – The translations are pulled into the website build directory.

The `downloadTranslationsFailSafe` variant tolerates failures for external pull requests that don't have access to the Crowdin environment token. Internal PRs have full access and use the non-failsafe variant.

## Phase 5: deployment

Deployment is handled by Netlify. Three Netlify build scripts cover the deployment contexts in `website/package.json`:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

### Production deployment

`netlify:build:production` runs the complete pipeline in order:

1. `docusaurus write-translations` – Writes the canonical English translation files.
2. `netlify:crowdin:delay` – Adds a delay to avoid Crowdin API rate limits.
3. `netlify:crowdin:uploadSources` – Uploads the fresh source strings to Crowdin.
4. `netlify:crowdin:downloadTranslations` – Waits for Crowdin and downloads the latest translations.
5. `build` – Runs the full production Docusaurus build with all locales.

### Preview and branch deployments

Deploy previews (`netlify:build:deployPreview`) and branch deploys (`netlify:build:branchDeploy`) both run a plain `pnpm build`. These environments skip the Crowdin workflow and build with the existing translation data.

### Local deployment testing

A local Netlify simulation is available:

```json
"netlify:test": "pnpm netlify:build:deployPreview && pnpm dlx --package netlify-cli netlify dev -- --debug"
```

This runs a deploy-preview build and starts the Netlify dev server with debug output.

### Docusaurus `deploy` command

The website exposes the standard Docusaurus deploy command for non-Netlify targets:

```json
"deploy": "docusaurus deploy"
```

The `@docusaurus/core` binary is defined in `packages/docusaurus/package.json`:

```json
"bin": {
  "docusaurus": "bin/docusaurus.mjs"
}
```

## Additional website test scripts

The website runs swizzle tests to verify theme component customization works in both eject and wrap modes, with and without TypeScript:

```json
"test:swizzle:eject:js": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:eject:ts": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:js": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:ts": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs"
```

The website also runs TypeScript type checking with:

```json
"typecheck": "tsc"
```

## Bundle profiling scripts

For performance analysis, two profiling scripts capture CPU profiles during the bundling phase of a Docusaurus build:

```json
"profile:bundle:cpu": "DOCUSAURUS_EXIT_AFTER_BUNDLING=true node --cpu-prof --cpu-prof-dir .cpu-prof ./node_modules/.bin/docusaurus build --locale en",
"profile:bundle:samply": "./profileSamply.sh"
```

The CPU profile is written to the `.cpu-prof` directory. The `profileSamply.sh` shell script is an alternative profiler invocation for the Samply profiler.

## Browserslist configuration

The website defines production and development browser targets in `website/package.json`:

```json
"browserslist": {
  "production": [
    ">0.5%",
    "not dead",
    "not op_mini all"
  ],
  "development": [
    "last 1 chrome version",
    "last 1 firefox version",
    "last 1 safari version"
  ]
}
```

## CI/CD dependency constraints

The pipeline depends on these runtime versions and package constraints:

- **Node.js**: `>=24.21` for all published packages (`@docusaurus/core`, `@docusaurus/lqip-loader`)
- **Playwright**: `^1.63.0` in the `argos` package
- **Argos CLI**: `^6.9.2` plus `@argos-ci/playwright` `^7.5.0`
- **Crowdin CLI**: `^5.0.2` and `@crowdin/crowdin-api-client` `^1.57.0`
- **Netlify plugin**: `netlify-plugin-cache` `^1.0.3`
- **React**: `^19.3.0` with matching `react-dom` for the website
- **Webpack**: `^5.111.0` across `website`, `@docusaurus/core`, and `@docusaurus/lqip-loader`

## Build artifacts and caching

The Netlify cache plugin (`netlify-plugin-cache`) is included in the website dependencies to persist build caches between Netlify builds. The `argos` upload script references screenshots stored at `./screenshots/chromium`.

The website `clear` script removes generated artifacts and test output:

```json
"clear": "docusaurus clear && rimraf changelog && rimraf _dogfooding/_swizzle_theme_tests"
```

## Local development commands

| Command | Location | Purpose |
| --- | --- | --- |
| `pnpm start` | `website/` | Start the dev server |
| `pnpm build` | `website/` | Production build |
| `pnpm serve` | `website/` | Serve a built site |
| `pnpm build` | each `packages/*/` | Compile TypeScript package |
| `pnpm watch` | each `packages/*/` | Watch and rebuild package |
| `pnpm screenshot` | `argos/` | Capture Playwright screenshots |
| `pnpm upload` | `argos/` | Upload screenshots to Argos |
| `pnpm report` | `argos/` | Open Playwright report |

## Environment variables

The pipeline uses the following environment variables:

| Variable | Used in | Purpose |
| --- | --- | --- |
| `BUILD_FAST` | `build:fast` scripts | Enables fast incremental builds |
| `RSDOCTOR` | `build:fast:rsdoctor` | Enables Rsdoctor bundle analysis |
| `BASE_URL` | `start:baseUrl`, `build:baseUrl` | Overrides the base URL to `/build/` |
| `SWIZZLE_ACTION` | Swizzle test scripts | Sets `eject` or `wrap` mode |
| `SWIZZLE_TYPESCRIPT` | Swizzle test scripts | Sets `'true'` or `'false'` |
| `DOCUSAURUS_EXIT_AFTER_BUNDLING` | `profile:bundle:cpu` | Exits after bundling for profiling |