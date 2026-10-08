# Test Strategy

This document describes the testing approach and coverage for the Docusaurus repository based on the scripts and dependencies declared in the prefetched package manifests.

## Testing toolkit

The repository uses two main testing areas:

- **Visual regression testing** through Argos and Playwright in `argos/package.json`
- **Website validation scripts** through `website/package.json`

The core packages use TypeScript build checks rather than a dedicated unit test runner in the prefetched source.

## Visual regression testing

The `argos` package runs Playwright screenshot tests and uploads the results to Argos for visual diff analysis.

`argos/package.json` declares these scripts:

| Script | Purpose |
| --- | --- |
| `screenshot` | Run Playwright tests that capture screenshots |
| `upload` | Upload screenshots from `./screenshots/chromium` to Argos |
| `upload-text-snapshots` | Upload the Docusaurus website build as text snapshots |
| `report` | Open the Playwright HTML report |

Argos dependencies include:

- `@argos-ci/cli` version `^6.9.2`
- `@argos-ci/playwright` version `^7.5.0`
- `@playwright/test` version `^1.63.0`
- `cheerio` version `^1.2.0`

The `upload-text-snapshots` script uploads the built website at `../website/build` with the build name `text-snapshots`. It includes `styles.css` and all HTML files under `docs/**` and `blog/**`.

## Website validation scripts

`website/package.json` contains several validation scripts that cover style ordering, theme swizzling, and TypeScript checks.

### CSS order validation

The `test:css-order` script runs `node testCSSOrder.mjs`.

This script validates that the generated CSS respects the expected ordering rules.

### Theme swizzle tests

The repository tests four swizzle combinations using environment variables:

| Script | `SWIZZLE_ACTION` | `SWIZZLE_TYPESCRIPT` |
| --- | --- | --- |
| `test:swizzle:eject:js` | `eject` | `false` |
| `test:swizzle:eject:ts` | `eject` | `true` |
| `test:swizzle:wrap:js` | `wrap` | `false` |
| `test:swizzle:wrap:ts` | `wrap` | `true` |

Each script invokes `node _dogfooding/testSwizzleThemeClassic.mjs` with the corresponding environment variables.

### TypeScript check

The `typecheck` script runs `tsc` in the `website` package.

This checks the TypeScript types used by the website and its dogfooding configuration.

## Build-based validation

`website/package.json` includes build variations that act as integration checks for different site configurations.

| Script | Purpose |
| --- | --- |
| `build` | Production build through `docusaurus build` |
| `build:baseUrl` | Build with `BASE_URL=/build/` |
| `build:blogOnly` | Build with `docusaurus.config-blog-only.js` |
| `build:fast` | Fast production build for the `en` locale only |
| `build:fast:rsdoctor` | Fast build with Rsdoctor bundle analysis enabled |

These scripts are not explicit test commands, but they verify that the website builds successfully under base URL, blog-only, fast, and Rsdoctor configurations.

## Core package build checks

Core packages use `tsc --build` as a compile-time validation layer.

`packages/docusaurus/package.json` declares:

- `build`: `tsc --build && node ../../admin/scripts/copyUntypedFiles.js`
- `watch`: `run-p -c copy:watch build:watch`

`packages/lqip-loader/package.json` declares:

- `build`: `tsc`
- `watch`: `tsc --watch`

These scripts do not run behavioral tests. They ensure that the packages compile successfully and that untyped files are copied after compilation.

## Fixture package

`admin/test-bad-package/package.json` declares a private package with intentionally old dependency versions:

- `@mdx-js/react` version `1.0.1`
- `react` version `16.14.0`
- `react-dom` version `16.14.0`

The prefetched files do not declare a script that runs this fixture. The package appears intended as a negative test fixture for dependency or peer dependency checks, but the exact runner is not present in the provided source context.

## Running tests locally

Run website validation commands from the `website` directory:

1. Change to the website directory:

   ```bash
   cd website
   ```

2. Run the CSS order check:

   ```bash
   pnpm test:css-order
   ```

3. Run all swizzle theme variations:

   ```bash
   pnpm test:swizzle:eject:js
   pnpm test:swizzle:eject:ts
   pnpm test:swizzle:wrap:js
   pnpm test:swizzle:wrap:ts
   ```

4. Run TypeScript checking:

   ```bash
   pnpm typecheck
   ```

Run visual regression tests from the `argos` directory:

1. Change to the Argos directory:

   ```bash
   cd argos
   ```

2. Capture screenshots:

   ```bash
   pnpm screenshot
   ```

3. Upload screenshots to Argos:

   ```bash
   pnpm upload
   ```

4. Open the Playwright report:

   ```bash
   pnpm report
   ```

Run core package build checks from their respective package directories:

```bash
cd packages/docusaurus
pnpm build
```

```bash
cd packages/lqip-loader
pnpm build
```

## Recommended test workflows

- For changes to website CSS or styling behavior, run `pnpm test:css-order`.
- For changes to theme swizzling, run all four `test:swizzle:*` scripts.
- For changes to website code or configuration, run `pnpm typecheck` and `pnpm build`.
- For visual changes, run `pnpm screenshot` and `pnpm upload` in `argos`.
- For core package changes, run the package `build` command to verify TypeScript compilation.

## Known limitations

The prefetched source context includes only package manifests. It does not include Playwright configuration files, test case files, Jest or Vitest configuration, CI workflow files, or test source files. Therefore, this document covers only the commands and dependencies visible in those manifests and does not describe individual test cases or assertions.