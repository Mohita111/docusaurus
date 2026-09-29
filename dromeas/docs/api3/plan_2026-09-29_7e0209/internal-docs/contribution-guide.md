# Contribution Guide

This guide describes how to set up and contribute to the Docusaurus monorepo. It is based on the package metadata in `website/package.json`, `packages/docusaurus/package.json`, `packages/lqip-loader/package.json`, `argos/package.json`, and `admin/test-bad-package/package.json`.

## Prerequisites

- Node.js `>=24.21`, as declared in `packages/docusaurus/package.json` and `packages/lqip-loader/package.json`
- pnpm, as indicated by the `workspace:*` dependency protocol and `pnpm` commands throughout `website/package.json`

## Workspace layout

| Directory | Package name | What it contains |
| --- | --- | --- |
| `packages/docusaurus` | `@docusaurus/core` | Core static site generator and CLI entry point |
| `packages/lqip-loader` | `@docusaurus/lqip-loader` | Webpack loader for low-quality image placeholders |
| `website` | `website` | Docusaurus documentation site and integration test harness |
| `argos` | `argos` | Visual diff tests using Playwright and Argos |
| `admin/test-bad-package` | `test-bad-package` | Private fixture with older React and MDX versions |

## Bootstrap the workspace

1. Install dependencies from the workspace root:

   ```bash
   pnpm install
   ```

2. Build the core package:

   ```bash
   pnpm --filter @docusaurus/core build
   ```

   The `@docusaurus/core` build script is:

   ```json
   "build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js"
   ```

3. Build the LQIP loader:

   ```bash
   pnpm --filter @docusaurus/lqip-loader build
   ```

   The `@docusaurus/lqip-loader` build script is:

   ```json
   "build": "tsc"
   ```

## Run the documentation site

The `website` package contains the main development site and test harness.

```bash
# Start the development server
pnpm --filter website start

# Create a production build
pnpm --filter website build

# Serve the production build locally
pnpm --filter website serve

# Clear generated files and dogfooding test output
pnpm --filter website clear
```

The underlying scripts in `website/package.json` are:

```json
"start": "docusaurus start",
"build": "docusaurus build",
"serve": "docusaurus serve",
"clear": "docusaurus clear && rimraf changelog && rimraf _dogfooding/_swizzle_theme_tests"
```

## Run development checks

### Type checking

Run the website TypeScript compiler:

```bash
pnpm --filter website typecheck
```

This invokes `tsc`, as defined in `website/package.json`.

### CSS order test

Run the CSS order verification script:

```bash
pnpm --filter website test:css-order
```

The underlying command is:

```json
"test:css-order": "node testCSSOrder.mjs"
```

### Swizzle compatibility tests

The `website` package defines four swizzle test variants. Each sets the `SWIZZLE_ACTION` and `SWIZZLE_TYPESCRIPT` environment variables before running the same dogfooding script.

```bash
pnpm --filter website test:swizzle:eject:js
pnpm --filter website test:swizzle:eject:ts
pnpm --filter website test:swizzle:wrap:js
pnpm --filter website test:swizzle:wrap:ts
```

For example, the eject JavaScript variant is:

```json
"test:swizzle:eject:js": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs"
```

## Run visual regression tests

The `argos` package manages visual diff tests.

```bash
# Run Playwright screenshot tests
pnpm --filter argos screenshot

# Open the Playwright HTML report
pnpm --filter argos report

# Upload Chromium screenshots to Argos
pnpm --filter argos upload
```

The package scripts are:

```json
"screenshot": "playwright test",
"upload": "pnpm exec -- argos upload ./screenshots/chromium",
"report": "playwright show-report"
```

The `upload-text-snapshots` script uploads built website text snapshots to Argos:

```json
"upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\""
```

## Translation and deployment scripts

Maintainers may use the Crowdin and Netlify scripts defined in `website/package.json`:

- `netlify:build:production`
- `netlify:build:branchDeploy`
- `netlify:build:deployPreview`
- `netlify:crowdin:downloadTranslations`
- `netlify:crowdin:uploadSources`

For example, the production Netlify build runs translation source upload and download before the site build:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build"
```

## Package metadata for contributors

The core package points to the public repository and issue tracker:

`packages/docusaurus/package.json`:

```json
"repository": {
  "type": "git",
  "url": "https://github.com/facebook/docusaurus.git",
  "directory": "packages/docusaurus"
},
"bugs": {
  "url": "https://github.com/facebook/docusaurus/issues"
}
```

`packages/lqip-loader/package.json`:

```json
"repository": {
  "type": "git",
  "url": "https://github.com/facebook/docusaurus.git",
  "directory": "packages/lqip-loader"
}
```

## Before submitting a pull request

Run the checks relevant to your change:

```bash
pnpm --filter @docusaurus/core build
pnpm --filter @docusaurus/lqip-loader build
pnpm --filter website typecheck
pnpm --filter website test:css-order
pnpm --filter website test:swizzle:eject:js
pnpm --filter website test:swizzle:eject:ts
pnpm --filter website test:swizzle:wrap:js
pnpm --filter website test:swizzle:wrap:ts
pnpm --filter argos screenshot
```

The provided source files do not include a root `package.json`, so any root-level scripts beyond the workspace commands shown here are not documented in this guide.