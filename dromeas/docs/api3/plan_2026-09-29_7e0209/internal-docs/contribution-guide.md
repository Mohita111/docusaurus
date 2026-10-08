# Contribution Guide

This guide covers how to set up, develop, and test the Docusaurus monorepo. All commands and paths in this document are sourced from the repository's `package.json` files.

## Prerequisites

Before contributing, install the following:

- **Node.js**: version `>=24.21`. This requirement is enforced in `packages/docusaurus/package.json` and `packages/lqip-loader/package.json` through the `engines` field.
- **pnpm**: The repository uses pnpm workspaces, evidenced by `workspace:*` dependency declarations throughout `website/package.json` and the `pnpm exec` and `pnpm --dir` commands in scripts.

## Repository structure

The repository is a monorepo organized into several top-level directories:

| Directory | Purpose |
|-----------|---------|
| `argos/` | Visual regression test suite using Playwright and Argos |
| `website/` | The Docusaurus documentation website (dogfooding target) |
| `packages/docusaurus/` | The `@docusaurus/core` package |
| `packages/lqip-loader/` | The `@docusaurus/lqip-loader` package for low-quality image placeholders |
| `admin/` | Administrative scripts and test fixtures, including `admin/test-bad-package/` |

The core package at `packages/docusaurus/package.json` defines:

- **Package name**: `@docusaurus/core`
- **Version**: `4.0.0`
- **License**: MIT
- **Entry point**: `bin/docusaurus.mjs` (the `docusaurus` binary)

## Set up the development environment

1. Clone the repository and install dependencies with pnpm:

   ```bash
   pnpm install
   ```

2. Verify the installation by building the core package:

   ```bash
   pnpm --dir packages/docusaurus build
   ```

   This runs `tsc --build` followed by `node ../../admin/scripts/copyUntypedFiles.js`.

## Development workflow

### Build the core package

Build `@docusaurus/core` from `packages/docusaurus/`:

```bash
pnpm build
```

This script, defined in `packages/docusaurus/package.json`, compiles TypeScript and copies untyped files:

```json
"build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js"
```

For watch mode during development:

```bash
pnpm watch
```

The `watch` script runs `copy:watch` and `build:watch` in parallel via `run-p`.

### Build the LQIP loader

From `packages/lqip-loader/`, build the package with:

```bash
pnpm build
```

The script runs `tsc` directly. Use `pnpm watch` for incremental compilation during development.

### Run the website locally

From `website/`, start the Docusaurus development server:

```bash
pnpm start
```

This runs `docusaurus start`. To build the production site:

```bash
pnpm build
```

To serve the built site locally:

```bash
pnpm serve
```

### Type checking

Run TypeScript type checking from the `website/` directory:

```bash
pnpm typecheck
```

The script is defined in `website/package.json` as `"typecheck": "tsc"`.

## Testing

### Visual regression tests

The `argos/` directory contains the visual diff test suite. Its `package.json` defines the following scripts:

| Script | Description |
|--------|-------------|
| `pnpm screenshot` | Runs Playwright tests to capture screenshots |
| `pnpm upload` | Uploads Chromium screenshots to Argos for comparison |
| `pnpm upload-text-snapshots` | Uploads built HTML text snapshots from `website/build` |
| `pnpm report` | Opens the Playwright HTML test report |

To run the visual tests:

```bash
cd argos
pnpm screenshot
```

Required dependencies for Argos testing, as listed in `argos/package.json`:

- `@argos-ci/cli` (^6.9.2)
- `@argos-ci/playwright` (^7.5.0)
- `@playwright/test` (^1.63.0)
- `cheerio` (^1.2.0)

### CSS order verification

The website includes a CSS ordering test that ensures styles load in the expected sequence:

```bash
cd website
pnpm test:css-order
```

This runs `node testCSSOrder.mjs` from the `website/` directory.

### Swizzle tests

Docusaurus supports theme swizzling (ejecting or wrapping theme components). The website package includes four test scripts that validate swizzle behavior with different combinations of action and TypeScript settings:

| Script | SWIZZLE_ACTION | SWIZZLE_TYPESCRIPT |
|--------|----------------|---------------------|
| `pnpm test:swizzle:eject:js` | `eject` | `false` |
| `pnpm test:swizzle:eject:ts` | `eject` | `true` |
| `pnpm test:swizzle:wrap:js` | `wrap` | `false` |
| `pnpm test:swizzle:wrap:ts` | `wrap` | `true` |

Each script sets environment variables with `cross-env` and runs `node _dogfooding/testSwizzleThemeClassic.mjs`.

## Build variants

The `website/package.json` defines several specialized build targets for testing different configurations:

### Base URL builds

Test the site with a custom base URL:

```bash
pnpm start:baseUrl
pnpm build:baseUrl
```

Both scripts set `BASE_URL='/build/'` with `cross-env` and pass it to `docusaurus start` or `docusaurus build`.

### Blog-only builds

Test solely the blog portion of the site:

```bash
pnpm start:blogOnly
pnpm build:blogOnly
```

These scripts use the `docusaurus.config-blog-only.js` configuration file via the `--config` flag.

### Fast builds

Skip expensive processing steps during development:

```bash
pnpm build:fast
```

This sets `BUILD_FAST=true` and builds only the `en` locale.

### RSdoctor bundle profiling

Analyze the bundle with RSdoctor:

```bash
pnpm build:fast:rsdoctor
```

This sets both `BUILD_FAST=true` and `RSDOCTOR=true` for a `en`-only build.

### CPU profiling

Profile the build bundling process:

```bash
pnpm profile:bundle:cpu
```

This sets `DOCUSAURUS_EXIT_AFTER_BUNDLING=true` and runs Node with `--cpu-prof`. Alternatively, use the Samply profiler script:

```bash
pnpm profile:bundle:samply
```

## Translation workflow

The website integrates with Crowdin for translations. The `netlify:crowdin:*` scripts in `website/package.json` manage the translation pipeline:

| Script | Purpose |
|--------|---------|
| `pnpm netlify:crowdin:downloadTranslations` | Wait for Crowdin and download translations |
| `pnpm netlify:crowdin:uploadSources` | Upload source files to Crowdin |
| `pnpm netlify:crowdin:wait` | Wait for Crowdin build completion |
| `pnpm netlify:crowdin:delay` | Introduce a delay for Crowdin processing |

To write translations locally from the `website/` directory:

```bash
pnpm write-translations
```

To regenerate heading IDs:

```bash
pnpm write-heading-ids
```

## Dependency validation

The `admin/test-bad-package/package.json` file serves as a test fixture for dependency compatibility validation. It intentionally declares incompatible versions:

- `@mdx-js/react`: `1.0.1` (the main repo expects `^3.1.1`)
- `react`: `16.14.0` (the main repo uses `^19.3.0`)
- `react-dom`: `16.14.0`

This fixture is used to verify that the monorepo correctly detects and reports peer dependency mismatches. Do not modify this package; its purpose is to trigger validation failures during testing.

## Code contributions checklist

Before submitting a change, complete these steps:

1. Run the build for any modified package (`pnpm build` in the affected package directory).
2. Run `pnpm typecheck` from `website/` to catch TypeScript errors.
3. If your change affects styles, run `pnpm test:css-order` from `website/`.
4. If your change affects themes or swizzling, run the relevant swizzle test script.
5. Run the Argos screenshot suite from `argos/` if visual output could be affected.
6. Verify your code adheres to the ESLint configuration from `@docusaurus/eslint-plugin` (declared as a dev dependency in `website/package.json`).

## Deployment targets

The `website/package.json` defines Netlify build scripts for different deployment contexts:

| Context | Script |
|---------|--------|
| Production | `pnpm netlify:build:production` |
| Branch deploy | `pnpm netlify:build:branchDeploy` |
| Deploy preview | `pnpm netlify:build:deployPreview` |

Local Netlify testing is available through:

```bash
pnpm netlify:test
```

This builds a deploy preview and serves it with `netlify dev`.

## Node.js engine requirement

All published packages enforce a minimum Node.js version of `24.21` through the `engines` field. This appears in both `packages/docusaurus/package.json` and `packages/lqip-loader/package.json`. Contributing to this codebase requires Node.js 24.21 or later.

## Limitations of this document

This guide is generated from the five `package.json` files available in the prefetched source set. The following aspects are not covered because the relevant files were not included:

- Detailed code style rules (ESLint configuration contents)
- Continuous integration configuration (GitHub Actions workflows)
- Commit message conventions
- Issue and pull request templates
- Testing methodologies for packages outside `argos/`, `website/`, and the core package

For these details, refer to the repository's `CONTRIBUTING.md`, `.github/` directory, and ESLint configuration files directly.