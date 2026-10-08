# Security policy for docusaurus

## Overview

This document describes security policies, access control mechanisms, and vulnerability reporting processes for the Docusaurus project (version 4.0.0). Docusaurus is an MIT-licensed open-source static site generator for building documentation websites.

The security posture documented here is derived from build configuration, dependency manifests, and deployment scripts present in the repository. Where a specific control is not verifiable from package manifests, the limitation is noted explicitly.

## Supported versions

| Version | Support status | Node.js requirement |
| --- | --- | --- |
| 4.0.0 | Current release | >=24.21 |

All packages in the monorepo declare a minimum Node.js version in their`engines` field:

```json
"engines": {
  "node": ">=24.21"
}
```

This requirement is enforced in the following packages:

-`@docusaurus/core` (packages/docusaurus/package.json)
-`@docusaurus/lqip-loader` (packages/lqip-loader/package.json)

Only the current major version receives security updates. Older versions are not supported for vulnerability patching.

## Reporting a vulnerability

To report a security issue in Docusaurus:

1. Do not open a public issue on GitHub.
2. Send details privately to the Docusaurus maintainers through the GitHub Security Advisory process at`https://github.com/facebook/docusaurus/security/advisories`.
3. Include a clear description of the vulnerability, affected versions, and reproduction steps.

Maintainers will acknowledge receipt and provide a timeline for triage and disclosure.

## Access control model

### Repository access

The repository is public under the MIT license, allowing read access to all source code. Write access is governed by GitHub's role-based access control and is managed by the Docusaurus maintainer team.

### Build-time secrets

Docusaurus uses **environment-variable-based secrets** for external service integration. Secrets are not stored in the repository or in package manifests. The`website/package.json` build scripts reference environment variables for credential handling:

| Script | Purpose | Secret exposure |
| --- | --- | --- |
|`netlify:crowdin:uploadSources` | Upload source strings to Crowdin | Requires`CROWDIN` credentials in environment |
|`netlify:crowdin:downloadTranslations` | Download translated content from Crowdin | Requires`CROWDIN` credentials in environment |
|`netlify:crowdin:downloadTranslationsFailSafe` | Download translations with fallback | Restricted to internal PRs only |

The`netlify:crowdin:downloadTranslationsFailSafe` script explicitly notes:

```
Crowdin translation download failure (only internal PRs have access to the Crowdin env token)
```

This confirms that **forked pull requests do not have access to the Crowdin environment token**. External contributions cannot exfiltrate Crowdin secrets through CI.

### CI/CD permissions

The deployment infrastructure supports branch-specific builds with differentiated permissions:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

Production builds execute Crowdin integration steps and require secret access, while branch and preview builds run only the standard site build without secrets.

## Dependency and supply chain security

### Workspace-local package resolution

Docusaurus uses **pnpm workspaces** with`workspace:*` protocol for internal packages. Internal packages are resolved locally from the monorepo rather than fetched from the npm registry:

```json
"dependencies": {
  "@docusaurus/core": "workspace:*",
  "@docusaurus/logger": "workspace:*",
  "@docusaurus/babel": "workspace:*",
  "@docusaurus/bundler": "workspace:*",
  "@docusaurus/mdx-loader": "workspace:*",
  "@docusaurus/utils": "workspace:*",
  "@docusaurus/utils-common": "workspace:*",
  "@docusaurus/utils-validation": "workspace:*",
  "@docusaurus/plugin-client-redirects": "workspace:*",
  "@docusaurus/plugin-content-blog": "workspace:*",
  "@docusaurus/plugin-content-docs": "workspace:*",
  "@docusaurus/plugin-content-pages": "workspace:*",
  "@docusaurus/plugin-google-gtag": "workspace:*",
  "@docusaurus/plugin-ideal-image": "workspace:*",
  "@docusaurus/plugin-pwa": "workspace:*",
  "@docusaurus/plugin-rsdoctor": "workspace:*",
  "@docusaurus/preset-classic": "workspace:*",
  "@docusaurus/theme-classic": "workspace:*",
  "@docusaurus/theme-common": "workspace:*",
  "@docusaurus/theme-live-codeblock": "workspace:*",
  "@docusaurus/theme-mermaid": "workspace:*"
}
```

This approach reduces supply chain risk for monorepo-internal packages because they are resolved locally and never retrieved from a potentially compromised registry.

### Third-party dependencies

Third-party dependencies are declared with caret and tilde ranges, allowing minor and patch updates:

```json
"dependencies": {
  "@crowdin/cli": "^5.0.2",
  "@crowdin/crowdin-api-client": "^1.57.0",
  "react": "^19.3.0",
  "react-dom": "^19.3.0",
  "webpack": "^5.111.0",
  "webpack-dev-server": "^6.0.0",
  "sharp": "^0.35.1"
}
```

The pnpm lockfile (not included in the provided source files) governs exact version resolution. For reproducible and auditable builds, commit the lockfile to version control and use it in CI environments.

### Dependencies with security exposure

The`website` package depends on Crowdin client libraries that handle API credentials:

-`@crowdin/cli ^5.0.2`
-`@crowdin/crowdin-api-client ^1.57.0`

Monitor security advisories for these packages. Ensure that credentials passed to these libraries are never logged or exposed in build output.

The core build pipeline depends on native and binary packages that require security updates:

-`sharp ^0.35.1`, Image processing; monitor for image parsing vulnerabilities
-`webpack ^5.111.0` and`webpack-dev-server ^6.0.0`, Bundling tools; update regularly

### Test fixtures with outdated dependencies

The`test-bad-package` fixture intentionally declares outdated React versions:

```json
"dependencies": {
  "@mdx-js/react": "1.0.1",
  "react": "16.14.0",
  "react-dom": "16.14.0"
}
```

This package is a test fixture used to validate version conflict handling and is **not shipped** to end users. Do not copy its dependency versions into production packages.

## Data protection

### Static site output

Docusaurus generates **static HTML, CSS, and JavaScript** files at build time. It does not run server-side code in production. There is no persistent server that stores user data.

### Runtime dependencies for tracking and PWA

The website includes optional plugins for analytics and offline functionality:

-`@docusaurus/plugin-google-gtag`, Google Analytics integration
-`@docusaurus/plugin-pwa`, Progressive Web App support

These plugins' configuration is in`docusaurus.config.js` (not provided in the source files). Review that configuration file to confirm:

- Whether Google Analytics is enabled and what data is collected
- Whether PWA service worker caching exposes sensitive documentation

### Personal data in translation workflows

Crowdin integration stores documentation source strings in the Crowdin cloud service. Do not include secrets, API keys, passwords, or personal data in documentation content intended for translation.

## Build and deployment security

### Build execution

The build is executed with:

```
pnpm build
```

This invokes`docusaurus build`, which uses Webpack 5 to bundle assets. Build-time security considerations:

- **Development server exposure**, Docusaurus includes`webpack-dev-server ^6.0.0` for local development. Never expose the dev server to untrusted networks if the site contains sensitive content.
- **Static file serving**, The core package includes`serve-handler ^6.1.7` for local development serving. Production deployments must use a properly configured static file host, not`serve-handler`.

### Visual regression testing

The`argos` directory contains visual diff testing via Playwright:

```json
"screenshot": "playwright test",
"upload": "pnpm exec -- argos upload ./screenshots/chromium"
```

The argos package depends on:

-`@argos-ci/cli ^6.9.2`
-`@argos-ci/playwright ^7.5.0`
-`@playwright/test ^1.63.0`

When uploading screenshots to Argos CI, ensure that only intended page snapshots are included. Do not run visual regression tests against environments that render secrets, API keys, or user-specific data.

### Base URL configuration

The site supports custom base URL configuration via environment variables:

```json
"start:baseUrl": "cross-env BASE_URL='/build/' pnpm start",
"build:baseUrl": "cross-env BASE_URL='/build/' pnpm build"
```

The base URL controls link and asset paths. Ensure that the deployment base URL is controlled by site administrators, not derived from user input, to prevent path traversal or link injection.

### Performance profiling and diagnostics

Build profiling scripts expose internal performance data:

```json
"profile:bundle:cpu": "DOCUSAURUS_EXIT_AFTER_BUNDLING=true node --cpu-prof --cpu-prof-dir .cpu-prof ./node_modules/.bin/docusaurus build --locale en"
```

CPU and bundle analysis output may contain sensitive information about site structure. Keep profiling results private and do not commit them to public repositories.

## Security of internal tooling

### Dynamic code evaluation

The core package (`@docusaurus/core`) depends on`eval ^0.1.8`:

```json
"dependencies": {
  "eval": "^0.1.8"
}
```

This package evaluates JavaScript at build time. Historically it is used for configuration evaluation. Any future use of`eval` with user-controllable input is a critical security risk. Restrict use to trusted, static configuration only.

### Image processing and LQIP generation

The`@docusaurus/lqip-loader` package processes images to generate low-quality image placeholders:

```json
"dependencies": {
  "sharp": "^0.35.1"
}
```

`sharp` is a native image processing library with a history of image parsing vulnerabilities. Keep`sharp` updated to the latest patch version and monitor its security advisories regularly.

### Utilities and validation

The core package includes utility packages for validation and common operations:

-`@docusaurus/utils-validation`, Configuration and input validation
-`@docusaurus/utils-common`, Shared utility functions

These packages are used during the build to parse user-supplied configuration. Monitor them for validation bypasses.

## Verification limitations

The following security controls are **not verifiable** from the provided package manifests and require additional repository files:

| Control | Requires file |
| --- | --- |
| Content Security Policy headers |`docusaurus.config.js`, hosting config |
| Dependency vulnerability scanning | CI workflow files (`.github/workflows`) |
| Code signing or artifact integrity verification | Release configuration |
| PWA service worker caching policy |`docusaurus.config.js` |
| Google Analytics and telemetry configuration |`docusaurus.config.js` |
| Crowdin integration authentication details |`docusaurus.config.js`, Netlify plugin config |
| Secure header configuration | Hosting provider config (Netlify, Vercel.) |

Review these files in the main repository to complete the security assessment.

## Policy enforcement

Docusaurus is a static site generator and does not include runtime authentication or authorization mechanisms. Security is enforced at the following layers:

1. **Repository access control**, GitHub role-based access and branch protection
2. **CI secret scoping**, Crowdin tokens unavailable to external fork PRs
3. **Dependency management**, workspace-local package resolution reduces registry risk
4. **Static output**, no server-side attack surface in production
5. **Hosting provider controls**, IP allowlisting, password protection, signed URLs

By default, Docusaurus sites are publicly readable. If you deploy a site with restricted content, configure access control at the hosting provider level (for example, Netlify password protection, AWS CloudFront signed URLs, or IP-based access control), not within the Docusaurus application layer.