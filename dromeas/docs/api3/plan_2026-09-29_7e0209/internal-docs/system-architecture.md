# System Architecture

Docusaurus is a monorepo that builds the `@docusaurus/core` static site generator, its official website, supporting webpack loaders, and testing infrastructure. This architecture overview is based on the package manifests for `packages/docusaurus`, `packages/lqip-loader`, `website`, `argos`, and `admin/test-bad-package`.

## Workspace model

The repository uses workspace linking. Internal dependencies use the `"workspace:*"` protocol in `package.json` files, for example in `packages/docusaurus/package.json`:

```json
"dependencies": {
  "@docusaurus/babel": "workspace:*",
  "@docusaurus/bundler": "workspace:*",
  "@docusaurus/logger": "workspace:*",
  "@docusaurus/mdx-loader": "workspace:*",
  "@docusaurus/utils": "workspace:*",
  "@docusaurus/utils-common": "workspace:*",
  "@docusaurus/utils-validation": "workspace:*"
}
```

Scripts invoke `pnpm` commands directly, for example `"upload": "pnpm exec -- argos upload ./screenshots/chromium"` in `argos/package.json`.

## Core package

`@docusaurus/core` is the main static site generator package. It provides the `docusaurus` command-line entry point:

```json
"bin": {
  "docusaurus": "bin/docusaurus.mjs"
}
```

Core is built with TypeScript:

```json
"scripts": {
  "build": "tsc --build && node ../../admin/scripts/copyUntypedFiles.js",
  "watch": "run-p -c copy:watch build:watch",
  "build:watch": "tsc --build --watch",
  "copy:watch": "node ../../admin/scripts/copyUntypedFiles.js --watch"
}
```

The dependency set shows the runtime responsibilities of `@docusaurus/core`:

| Responsibility | Related dependencies |
| --- | --- |
| Bundling and dev server | `webpack`, `webpack-dev-server`, `webpack-merge`, `webpack-bundle-analyzer` |
| Routing | `react-router`, `react-router-config`, `react-router-dom` |
| Server-side rendering and code splitting | `react-loadable`, `react-loadable-ssr-addon-v5-slorber`, `react-helmet-async` |
| CLI parsing and prompts | `commander`, `prompts` |
| File watching and static serving | `chokidar`, `serve-handler` |
| MDX processing | `@docusaurus/mdx-loader` |
| Logging | `@docusaurus/logger` |
| Validation and shared utilities | `@docusaurus/utils`, `@docusaurus/utils-common`, `@docusaurus/utils-validation` |

All internal packages in the provided manifests use version `4.0.0` and require Node.js `>=24.21`.

## Website layer

The `website` package is the Docusaurus documentation site. It consumes many Docusaurus packages through workspace dependencies:

```json
"dependencies": {
  "@docusaurus/core": "workspace:*",
  "@docusaurus/preset-classic": "workspace:*",
  "@docusaurus/theme-classic": "workspace:*",
  "@docusaurus/theme-common": "workspace:*",
  "@docusaurus/plugin-content-blog": "workspace:*",
  "@docusaurus/plugin-content-docs": "workspace:*",
  "@docusaurus/plugin-content-pages": "workspace:*",
  "@docusaurus/plugin-client-redirects": "workspace:*",
  "@docusaurus/plugin-google-gtag": "workspace:*",
  "@docusaurus/plugin-ideal-image": "workspace:*",
  "@docusaurus/plugin-pwa": "workspace:*",
  "@docusaurus/plugin-rsdoctor": "workspace:*",
  "@docusaurus/remark-plugin-npm2yarn": "workspace:*",
  "@docusaurus/theme-live-codeblock": "workspace:*",
  "@docusaurus/theme-mermaid": "workspace:*",
  "@docusaurus/types": "workspace:*"
}
```

This package demonstrates the Docusaurus architecture in practice: `@docusaurus/core` orchestrates content plugins and themes, while `@docusaurus/preset-classic` groups the default blog, docs, and pages plugins together.

The `website/package.json` scripts show the main build and development workflows:

- `"start": "docusaurus start"`
- `"build": "docusaurus build"`
- `"serve": "docusaurus serve"`
- `"deploy": "docusaurus deploy"`
- `"clear": "docusaurus clear && rimraf changelog && rimraf _dogfooding/_swizzle_theme_tests"`

Additional scripts support dogfooding and performance profiling:

```json
"test:swizzle:eject:js": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='false' node _dogfooding/testSwizzleThemeClassic.mjs",
"test:swizzle:eject:ts": "cross-env SWIZZLE_ACTION='eject' SWIZZLE_TYPESCRIPT='true' node _dogfooding/testSwizzleThemeClassic.mjs",
"build:fast": "cross-env BUILD_FAST=true pnpm build --locale en",
"build:fast:rsdoctor": "cross-env BUILD_FAST=true RSDOCTOR=true pnpm build --locale en"
```

The website uses `@docusaurus/plugin-rsdoctor` for bundle analysis and `@docusaurus/theme-mermaid` and `@docusaurus/theme-live-codeblock` for enhanced content rendering.

## Content and theme composition

The website depends on separate content plugins for docs, blog, and pages. This reflects the separation of concerns in the Docusaurus architecture:

| Package | Role |
| --- | --- |
| `@docusaurus/plugin-content-docs` | Loads and builds Markdown documentation |
| `@docusaurus/plugin-content-blog` | Loads and builds blog posts |
| `@docusaurus/plugin-content-pages` | Builds standalone React pages |
| `@docusaurus/theme-classic` | Provides the default theme |
| `@docusaurus/theme-common` | Shared theme utilities |
| `@docusaurus/theme-live-codeblock` | Live code block rendering |
| `@docusaurus/theme-mermaid` | Mermaid diagram rendering |

The core package depends on `@docusaurus/mdx-loader`, which provides MDX processing for these content plugins.

## Supporting loaders

`@docusaurus/lqip-loader` is a webpack loader for generating low-quality image placeholders:

```json
{
  "name": "@docusaurus/lqip-loader",
  "description": "Low Quality Image Placeholders (LQIP) loader for webpack.",
  "main": "lib/index.js",
  "dependencies": {
    "@docusaurus/logger": "workspace:*",
    "file-loader": "^6.2.0",
    "lodash": "^4.18.1",
    "sharp": "^0.35.1",
    "tslib": "^2.8.1"
  }
}
```

It uses `sharp` for image processing and `file-loader` to handle emitted placeholder files. The build script is `tsc`, producing CommonJS output at `lib/index.js`.

## Visual regression testing

The `argos` package provides visual diff testing for the Docusaurus website:

```json
"scripts": {
  "screenshot": "playwright test",
  "upload": "pnpm exec -- argos upload ./screenshots/chromium",
  "upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\"",
  "report": "playwright show-report"
}
```

This package has two jobs:

1. Run Playwright screenshot tests against the site.
2. Upload screenshots or built HTML snapshots to Argos for visual regression comparison.

Dependencies include `@argos-ci/cli`, `@argos-ci/playwright`, and `@playwright/test`. The `cheerio` dependency supports HTML text-snapshot processing.

## Dependency validation fixture

`admin/test-bad-package` contains intentionally invalid dependency versions:

```json
"dependencies": {
  "@mdx-js/react": "1.0.1",
  "react": "16.14.0",
  "react-dom": "16.14.0"
}
```

The rest of the monorepo targets React `19.3.0` and `@mdx-js/react` `^3.1.1`. This fixture is used to test dependency constraint validation and to verify that incompatible dependency combinations are detected.

## Deployment architecture

The `website/package.json` defines Netlify deployment scripts that show the translation and build flow:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build",
"netlify:test": "pnpm netlify:build:deployPreview && pnpm dlx --package netlify-cli netlify dev -- --debug"
```

Production builds write translations, coordinate with Crowdin through helper scripts, then run `docusaurus build`. Branch deploy and preview builds run `docusaurus build` directly.

## Runtime constraints

All provided package manifests enforce the same Node.js engine:

```json
"engines": {
  "node": ">=24.21"
}
```

The website config also defines browser targets:

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