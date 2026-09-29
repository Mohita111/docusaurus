# Test Strategy

This document describes the testing and validation approach for the Docusaurus 4.0.0 monorepo, based on the package manifests and scripts available in the repository. The strategy combines visual regression testing, website dogfooding tests, TypeScript validation, build-variant checks, and dependency fixture validation.

## Testing layers

### 1. Visual regression tests with Argos and Playwright

The `argos` package provides visual diff tests that run against the generated website.

**Primary source:** `argos/package.json`

Available scripts:

| Script | Command | Purpose |
| --- | --- | --- |
| `screenshot` | `playwright test` | Runs Playwright tests that capture screenshots into `./screenshots/chromium`. |
| `upload` | `pnpm exec -- argos upload ./screenshots/chromium` | Uploads Chromium screenshots to Argos for visual comparison. |
| `upload-text-snapshots` | `pnpm exec -- argos upload "../website/build" --build-name text-snapshots --files styles.css --files "docs/**/*.html" --files "blog/**/*.html"` | Uploads production HTML and CSS snapshots for text-level diffing. |
| `report` | `playwright show-report` | Opens the local Playwright HTML report. |

The visual test dependencies are:

- `@argos-ci/cli` version `^6.9.2`
- `@argos-ci/playwright` version `^7.5.0`
- `@playwright/test` version `^1.63.0`
- `cheerio` version `^1.2.0`

The `upload-text-snapshots` command explicitly targets:

- `styles.css`
- All HTML files under `docs/**/*.html`
- All HTML files under `blog/**/*.html`

This separates structural or content regressions in documentation pages from pixel-level visual regressions.

### 2. Website behavior and theme tests

The `website` package contains specialized behavior tests and TypeScript validation.

**Primary source:** `website/package.json`

#### CSS order validation

```json
"test:css-order": "node testCSSOrder.mjs"
```

This script executes `testCSSOrder.mjs` directly with Node.js. It validates that CSS files are emitted in the expected order after the Docusaurus build.

#### Theme swizzle matrix

The website runs four dogfooding tests that exercise theme swizzle combinations:

```json
"test:swizzle:eject:js": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:eject:ts": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:js": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:ts": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs"
```

The coverage matrix is:

| Swizzle action | JavaScript mode | TypeScript mode |
| --- | --- | --- |
| `eject` | `test:swizzle:eject:js` | `test:swizzle:eject:ts` |
| `wrap` | `test:swizzle:wrap:js` | `test:swizzle:wrap:ts` |

Each command uses the shared test runner `_dogfooding/testSwizzleThemeClassic.mjs` and sets two environment variables:

- `SWIZZLE_ACTION`: either `eject` or `wrap`
- `SWIZZLE_TYPESCRIPT`: either `false` or `true`

The `cross-env` package ensures environment variables work consistently across platforms.

#### TypeScript validation

```json
"typecheck": "tsc"
```

The website package runs `tsc` directly to catch TypeScript errors in the dogfooding website configuration and supporting scripts.

### 3. Build-based validation

Build variants in `website/package.json` verify that Docusaurus can produce a site under different configurations.

**Primary source:** `website/package.json`

| Script | Purpose |
| --- | --- |
| `build` | Runs the standard `docusaurus build`. |
| `build:fast` | Runs `docusaurus build --locale en` with `BUILD_FAST=true` to exercise the fast build path. |
| `build:fast:rsdoctor` | Runs the fast build with `RSDOCTOR=true` to enable Rsdoctor bundle diagnostics. |
| `build:baseUrl` | Runs a build with `BASE_URL='/build/'` to verify custom base URL handling. |
| `build:blogOnly` | Runs a build using `docusaurus.config-blog-only.js` to test a blog-only configuration. |
| `start:baseUrl` | Starts the dev server with `BASE_URL='/build/'` for local base URL testing. |
| `start:blogOnly` | Starts the dev server with the blog-only config. |

Bundle profiling scripts also serve as performance validation:

```json
"profile:bundle:cpu": "DOCUSAURUS_EXIT_AFTER_BUNDLING=true node --cpu-prof --cpu-prof-dir .cpu-prof ./node_modules/.bin/docusaurus build --locale en",
"profile:bundle:samply": "./profileSamply.sh"
```

These scripts exit after bundling and capture CPU profiles for bundle performance analysis.

### 4. Netlify deployment validation

The website includes Netlify-specific scripts for validating production, branch, and preview builds.

**Primary source:** `website/package.json`

```json
"netlify:test": "pnpm netlify:build:deployPreview && pnpm dlx --package netlify-cli netlify dev -- --debug"
```

This command:

1. Builds a deploy preview through `netlify:build:deployPreview`.
2. Runs a local Netlify dev server with the `--debug` flag.

Production and preview build paths are covered by:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

The production path additionally validates Crowdin translation upload and download, with a fail-safe variant:

```json
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')"
```

### 5. Package compile and copy validation

Core packages use TypeScript compilation as a build-time validation gate.

**Primary source:** `packages/docusaurus/package.json`

```json
"build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js"
```

The core package build runs:

1. `tsc --build` for project-level TypeScript compilation.
2. `copyUntypedFiles.js` to copy non-TypeScript assets that are not emitted by the compiler.

Watch mode splits these into two parallel tasks:

```json
"watch": "run-p -c copy:watch build:watch",
"build:watch": "tsc --build --watch",
"copy:watch": "node ../../admin/scripts/copyUntypedFiles.js --watch"
```

The `lqip-loader` package uses a simpler compile gate:

**Primary source:** `packages/lqip-loader/package.json`

```json
"build": "tsc",
"watch": "tsc --watch"
```

### 6. Dependency compatibility fixture

The `admin/test-bad-package` directory contains a private fixture package with intentionally inconsistent dependency versions.

**Primary source:** `admin/test-bad-package/package.json`

```json
{
  "name": "test-bad-package",
  "version": "4.0.0",
  "private": true,
  "dependencies": {
    "@mdx-js/react": "1.0.1",
    "react": "16.14.0",
    "react-dom": "16.14.0"
  }
}
```

This fixture pins:

- `@mdx-js/react` to `1.0.1`
- `react` to `16.14.0`
- `react-dom` to `16.14.0`

The provided files do not include the consuming script or verification logic. The fixture is evidence of a dependency-validation scenario, but its exact execution path is not present in the source context.

## Coverage summary

| Test area | Entry point | Execution model |
| --- | --- | --- |
| Visual regression screenshots | `argos/package.json` | Playwright + Argos |
| Text and HTML snapshots | `argos/package.json` | Argos upload from production build |
| CSS ordering | `website/package.json` | Node script |
| Theme swizzle combinations | `website/package.json` | 2x2 matrix with dogfooding runner |
| Website TypeScript | `website/package.json` | `tsc` |
| Standard and fast builds | `website/package.json` | `docusaurus build` variants |
| Base URL and blog-only builds | `website/package.json` | `docusaurus build` with custom config |
| Bundle performance profiling | `website/package.json` | Node CPU profiler and Samply |
| Netlify deploy preview | `website/package.json` | `netlify dev --debug` |
| Core package compilation | `packages/docusaurus/package.json` | `tsc --build` + file copy |
| LQIP loader compilation | `packages/lqip-loader/package.json` | `tsc` |
| Bad dependency fixture | `admin/test-bad-package/package.json` | Fixture only, consuming harness not provided |

## Limitations of the provided source context

The supplied package manifests do not include dedicated unit test scripts such as Jest or Vitest. The `packages/docusaurus/package.json` provides build and watch scripts, but no explicit test script. The exact Playwright configuration, Argos comparison thresholds, and the contents of `testCSSOrder.mjs` or `testSwizzleThemeClassic.mjs` are not included in the provided files. Any documentation of those internals would require the corresponding source files.