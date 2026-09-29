# CI/CD Pipeline

## Overview

The Docusaurus repository uses a multi-stage CI/CD pipeline distributed across npm package scripts, Netlify for website deployment, Crowdin for translation management, and Argos with Playwright for visual regression testing. This document describes the pipeline stages, the scripts that drive them, and the dependencies between them.

The build target for Node.js is fixed via the `engines` field in `packages/docusaurus/package.json` and `packages/lqip-loader/package.json`:

```json
"engines": {
  "node": ">=24.21"
}
```

---

## Build pipeline

The repository builds individual packages with TypeScript before the website artifact can be produced.

### Core package build

The `@docusaurus/core` package in `packages/docusaurus/package.json` compiles TypeScript and then copies untyped files:

```json
"scripts": {
  "build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js",
  "watch": "run-p -c copy:watch build:watch",
  "build:watch": "tsc --build --watch",
  "copy:watch": "node ../../admin/scripts/copyUntypedFiles.js --watch"
}
```

The `build` script runs two commands in sequence:

1. `tsc --build` compiles the TypeScript sources.
2. `node ../../admin/scripts/copyUntypedFiles.js` copies non-TypeScript files (such as templates and static assets) into the build output.

### LQIP loader package build

The `@docusaurus/lqip-loader` package in `packages/lqip-loader/package.json` uses a plain TypeScript compile:

```json
"scripts": {
  "build": "tsc",
  "watch": "tsc --watch"
}
```

---

## Website build

The main website artifact is built by the `website` package in `website/package.json`.

### Standard production build

```json
"scripts": {
  "build": "docusaurus build"
}
```

### Fast build for English-only iterations

```json
"build:fast": "cross-env BUILD_FAST=true pnpm build --locale en"
```

This command sets the `BUILD_FAST` environment variable and restricts the build to the `en` locale, which reduces build time during local iteration.

### Fast build with Rsdoctor bundle analysis

```json
"build:fast:rsdoctor": "cross-env BUILD_FAST=true RSDOCTOR=true pnpm build --locale en"
```

This command additionally enables `RSDOCTOR`, which triggers the `@docusaurus/plugin-rsdoctor` dependency declared in `website/package.json`.

### Base URL build

```json
"build:baseUrl": "cross-env BASE_URL='/build/' pnpm build"
```

This command builds the site with a custom base URL of `/build/`, useful for preview environments.

---

## Netlify deployment pipeline

The `website` package defines three Netlify entry points in `website/package.json`, each selected by Netlify depending on the deploy context.

### Production deployment

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build"
```

The production pipeline runs five stages in order:

1. `pnpm docusaurus write-translations` generates the translation source files for all locales.
2. `pnpm netlify:crowdin:delay` runs `node delayCrowdin.mjs` to wait before interacting with Crowdin.
3. `pnpm netlify:crowdin:uploadSources` runs `pnpm --dir .. crowdin:upload:website` to upload new translation sources.
4. `pnpm netlify:crowdin:downloadTranslations` runs `pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website` to wait for Crowdin processing and download translated content.
5. `pnpm build` produces the final static site.

### Branch deploys and deploy previews

```json
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

Non-production Netlify contexts skip the Crowdin steps and run only the standard Docusaurus build.

### Local Netlify testing

```json
"netlify:test": "pnpm netlify:build:deployPreview && pnpm dlx --package netlify-cli netlify dev -- --debug"
```

This command builds the deploy preview and then serves it through `netlify-cli` in debug mode.

---

## Crowdin translation workflow

The Crowdin integration is driven through the root repository. The individual npm scripts in `website/package.json` are:

```json
"netlify:crowdin:delay": "node delayCrowdin.mjs",
"netlify:crowdin:wait": "node waitForCrowdin.mjs",
"netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website",
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')",
"netlify:crowdin:uploadSources": "pnpm --dir .. crowdin:upload:website"
```

The `downloadTranslationsFailSafe` variant is specifically designed for fork pull requests and contributor environments that do not have access to the Crowdin API token. It logs a fallback message instead of failing the build:

> Crowdin translation download failure (only internal PRs have access to the Crowdin env token)

The Crowdin CLI is available as a depdency in `website/package.json`:

```json
"@crowdin/cli": "^5.0.2",
"@crowdin/crowdin-api-client": "^1.57.0"
```

---

## Visual regression testing with Argos

The `argos` package in `argos/package.json` handles visual diff testing using Playwright and the Argos CLI.

```json
"scripts": {
  "screenshot": "playwright test",
  "upload": "pnpm exec -- argos upload ./screenshots/chromium",
  "upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\"",
  "report": "playwright show-report"
}
```

The relevant dependencies are:

```json
"dependencies": {
  "@argos-ci/cli": "^6.9.2",
  "@argos-ci/playwright": "^7.5.0",
  "@playwright/test": "^1.63.0"
}
```

The workflow is:

1. `screenshot` runs the Playwright test suite, which captures screenshots of pages using the `@argos-ci/playwright` integration and writes them to `./screenshots/chromium`.
2. `upload` sends the screenshot directory to Argos for comparison against the baseline.
3. `upload-text-snapshots` uploads the built website HTML and CSS files from `../website/build` (the output of `pnpm build` in the `website` package) for text-only snapshot comparisons.
4. `report` opens the Playwright HTML report locally for debugging failures.

---

## Type checking and static tests

The `website` package runs a standalone TypeScript type check:

```json
"typecheck": "tsc"
```

The website package also includes CSS ordering and swizzle tests:

```json
"test:css-order": "node testCSSOrder.mjs",
"test:swizzle:eject:js": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:eject:ts": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:js": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:wrap:ts": "cross-env SWIZZLE_ACTION='wrap' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs"
```

The swizzle tests exercise two actions (`eject` and `wrap`) across JavaScript and TypeScript variants, controlled by the `SWIZZLE_ACTION` and `SWIZZLE_TYPESCRIPT` environment variables.

---

## Runtime requirements

Both `@docusaurus/core` and `@docusaurus/lqip-loader` declare:

```json
"engines": {
  "node": ">=24.21"
}
```

Any CI runner that builds or tests these packages must use Node.js version 24.21 or later.

---

## Documentation limitations

The following details are not verifiable from the provided source files and are therefore not documented here:

- The exact CI platform configuration files (for example, GitHub Actions workflows such as `.github/workflows/*.yml`, CircleCI config, or equivalent) were not included.
- The `netlify.toml` configuration that maps Netlify contexts to the `netlify:*` build commands is not present in the provided files.
- Environment variables and secrets (such as `CROWDIN_TOKEN`, `ARGOS_TOKEN`, and Netlify API credentials) are referenced implicitly by the scripts but are not defined in the provided `package.json` files.
- Monorepo orchestration scripts referenced by the website scripts (for example, the root `crowdin:upload:website` and `crowdin:download:website` scripts) are defined in the root `package.json`, which was not included.

If you need to document those areas, include the root `package.json`, the CI workflow files, and `netlify.toml` in the source set.