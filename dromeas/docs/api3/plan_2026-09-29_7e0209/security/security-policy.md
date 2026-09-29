# Security Policy

## Overview

This document outlines the security posture, access controls, and security-relevant configurations for the Docusaurus repository as derived from the package manifests in the codebase. It covers package visibility, runtime constraints, sensitive build operations, dependency surfaces, and deployment security considerations.

Docusaurus is an open source documentation website generator distributed under the MIT license. The repository contains both public npm packages and private workspace packages used for internal testing and the project website.

## Package access controls

The repository distinguishes between public and private packages through the `private`, `publishConfig`, and `license` fields in each `package.json`.

### Public packages

| Package | Visibility | Access | License | Publish root |
|---|---|---|---|---|
| `@docusaurus/core` | Public | `public` | MIT | `packages/docusaurus` |
| `@docusaurus/lqip-loader` | Public | `public` | MIT | `packages/lqip-loader` |

The `@docusaurus/core` manifest sets `publishConfig.access` to `public`:

```json
"publishConfig": {
  "access": "public"
}
```

The same configuration applies to `@docusaurus/lqip-loader`. Both packages declare `"license": "MIT"`, establishing the open source distribution terms.

### Private workspace packages

The following packages are marked `private: true` and are not published to npm:

- `website` — version `4.0.0`, the project documentation website
- `argos` — version `4.0.0`, Argos visual diff test package
- `test-bad-package` — version `4.0.0`, test fixture for invalid dependency versions

Private packages retain MIT licensing declarations where present (`argos` declares `"license": "MIT"`; `website` omits the field but inherits the repository license).

## Runtime environment requirements

All public packages declare a minimum Node.js runtime:

```json
"engines": {
  "node": ">=24.21"
}
```

This constraint appears in both `@docusaurus/core` and `@docusaurus/lqip-loader`. The website package does not explicitly declare an `engines` block in the provided manifest.

## Sensitive operations and token handling

### Crowdin translation tokens

The `website/package.json` contains build scripts that interact with Crowdin for translation management. The script names and composition reveal token access patterns:

```json
"netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website",
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')",
"netlify:crowdin:uploadSources": "pnpm --dir .. crowdin:upload:website"
```

The `netlify:crowdin:downloadTranslationsFailSafe` script embeds a fallback message confirming that Crowdin environment tokens are restricted to internal pull requests. External PRs from forks do not have access to the Crowdin token, and the script fails safe with an echo message rather than aborting the build.

The token is consumed by `@crowdin/cli` version `^5.0.2` and `@crowdin/crowdin-api-client` version `^1.57.0`, both declared as website dependencies. The token itself is supplied through environment variables during Netlify builds; no hardcoded credentials appear in the package manifests.

### Netlify deployment scripts

The `website/package.json` defines three Netlify build variants:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

The production build sequence runs Crowdin upload and download steps, meaning production deployments require translation tokens. Branch and deploy preview builds skip Crowdin steps entirely, reducing token exposure for non-production environments.

### Visual diff testing tokens

The `argos/package.json` uses `@argos-ci/cli` version `^6.9.2` and `@argos-ci/playwright` version `^7.5.0` for visual regression testing:

```json
"scripts": {
  "upload": "pnpm exec -- argos upload ./screenshots/chromium",
  "upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\""
}
```

The `upload` scripts send artifacts to the Argos CI service. Authentication uses environment-managed tokens from the Argos CLI; no tokens are embedded in the manifest.

## Dependency security surface

### Core package dependencies

`@docusaurus/core` includes the following security-relevant dependencies:

| Dependency | Version range | Security relevance |
|---|---|---|
| `escape-html` | `^1.0.3` | HTML escaping to prevent XSS in generated output |
| `eval` | `^0.1.8` | Dynamic code evaluation during build, requires careful input validation |
| `serve-handler` | `^6.1.7` | Local static file server for `docusaurus serve` |
| `webpack-dev-server` | `^6.0.0` | Development server with hot reload capabilities |
| `react-helmet-async` | npm alias `@slorber/react-helmet-async@1.3.0` | Manage document head tags safely in React |
| `commander` | `^15.0.0` | CLI argument parsing |

The `eval` dependency warrants particular scrutiny. It is used at build time for dynamic evaluation and must not be exposed to untrusted input. The `escape-html` dependency provides XSS protection for server-rendered content.

### Image processing dependencies

`@docusaurus/lqip-loader` uses `sharp` version `^0.35.1` for image processing:

```json
"dependencies": {
  "sharp": "^0.35.1"
}
```

Sharp processes untrusted image inputs during builds. Version `^0.35.1` includes native image decoding; security advisories for sharp should be monitored regularly.

### Website plugin dependencies

The website depends on several Docusaurus plugins with security implications:

- `@docusaurus/plugin-pwa` — implements service worker behavior, requiring careful cache invalidation
- `@docusaurus/plugin-client-redirects` — handles HTTP redirects, requiring validation of redirect targets
- `@docusaurus/plugin-google-gtag` — loads Google Analytics scripts, introducing third-party script execution
- `@docusaurus/theme-live-codeblock` — executes code examples in the browser

Additionally, the website uses `react-lite-youtube-embed` version `^3.7.0` for embedding YouTube content, which loads third-party iframes.

### Test fixture package

The `admin/test-bad-package/package.json` deliberately pins outdated dependencies:

```json
"dependencies": {
  "@mdx-js/react": "1.0.1",
  "react": "16.14.0",
  "react-dom": "16.14.0"
}
```

This package exists as a test fixture, not as production code. React 16.14.0 and MDX React 1.0.1 are end-of-life versions with known security advisories. This fixture must never be published or used in production builds; its `private: true` flag enforces this.

## Browser support policy

The website declares distinct browser targets for production and development:

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

The production configuration targets browsers with at least 0.5% market share that are actively maintained. The `not op_mini all` exclusion removes Opera Mini support, reflecting that modern web platform features used by Docusaurus 4.0.0 require browser capabilities not present in Opera Mini.

## Dependency lock and supply chain considerations

All workspace dependencies use the `workspace:*` protocol, pinning intra-repository packages to exact versions from the workspace. This prevents accidental use of mismatched package versions from npm during development.

External dependencies use caret (`^`) ranges, which allow minor and patch updates. This is the default npm behavior and relies on lock files for reproducible installs. The repository packages do not contain bundled lock files in the provided manifests.

Peer dependencies in `@docusaurus/core` require `react` and `react-dom` at `^19.3.0`:

```json
"peerDependencies": {
  "@mdx-js/react": "^3.1.1",
  "react": "^19.3.0",
  "react-dom": "^19.3.0"
}
```

The `@docusaurus/faster` peer dependency is marked optional:

```json
"peerDependenciesMeta": {
  "@docusaurus/faster": {
    "optional": true
  }
}
```

## Security practices derived from build scripts

### Production build isolation

The `website/package.json` includes a `netlify:build:production` script that runs translations synchronization before the standard build. This sequence ensures that production deployments include localized content while branch deploys and previews use simplified builds, limiting access to production-only resources.

### Type checking

The website runs TypeScript as a static analysis step:

```json
"typecheck": "tsc"
```

Type checking is not a security control per se, but it provides an additional validation layer during development that can catch type-related vulnerabilities before deployment.

### CSS order testing

The `test:css-order` script validates CSS output ordering:

```json
"test:css-order": "node testCSSOrder.mjs"
```

This is a correctness check, not a security control, but ensures deterministic output required for subresource integrity validation.

## Documented limitations

This document is derived from five package manifests in the repository: `argos/package.json`, `website/package.json`, `packages/docusaurus/package.json`, `packages/lqip-loader/package.json`, and `admin/test-bad-package/package.json`. The following security-relevant information is not available in these files and requires source code inspection:

- A standalone `SECURITY.md` file was not among the provided sources. The repository's official security reporting process and vulnerability disclosure policy are not documented here.
- The `website/package.json` references a `profileSamply.sh` script that is not provided; its contents and any security implications cannot be assessed.
- The Crowdin and Argos token acquisition mechanisms (OAuth scopes, token permissions, rotation policies) are not documented in the manifests.
- The `netlify-plugin-cache` dependency version `^1.0.3` is declared, but its cache invalidation behavior and any security controls are not specified.
- Content Security Policy (CSP) headers for the website are not configured in these manifests; CSP would typically be set through Netlify configuration or Docusaurus plugin configuration.