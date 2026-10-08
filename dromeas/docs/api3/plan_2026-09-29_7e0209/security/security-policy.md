# Security Policy for Docusaurus

## Overview

This document describes security policies, access control mechanisms, and vulnerability reporting processes for the Docusaurus project (version 4.0.0). Docusaurus is an MIT-licensed open-source static site generator for building documentation websites.

The security posture documented here is derived from build configuration, dependency manifests, and deployment scripts present in the repository. Where a specific control is not verifiable from package manifests, the limitation is noted explicitly.

## Supported versions

| Version | Support status | Node.js requirement |
| --- | --- | --- |
| 4.0.0 | Current release | >=24.21 |

All packages in the monorepo declare:

```json
"engines": {
  "node": ">=24.21"
}
```

Only the current major version receives security updates. Older versions are not supported for vulnerability patching.

## Reporting a vulnerability

To report a security issue in Docusaurus:

1. Do not open a public issue on GitHub.
2. Send details privately to the Docusaurus maintainers through the GitHub Security Advisory process at `https://github.com/facebook/docusaurus/security/advisories`.
3. Include a clear description of the vulnerability, affected versions, and reproduction steps.

Maintainers will acknowledge receipt and provide a timeline for triage and disclosure.

## Access control model

### Repository access

The repository is public under the MIT license, allowing read access to all source code. Write access is governed by GitHub's role-based access control and is managed by the Docusaurus maintainer team.

### Build-time secrets

Docusaurus uses **environment-variable-based secrets** for external service integration. Secrets are not stored in the repository or in package manifests.

The following build scripts reference environment variables and tokens:

| Script | Purpose | Secret exposure |
| --- | --- | --- |
| `netlify:crowdin:uploadSources` | Upload source strings to Crowdin | Requires `CROWDIN` credentials in environment |
| `netlify:crowdin:downloadTranslations` | Download translated content from Crowdin | Requires `CROWDIN` credentials in environment |

The Crowdin token access is restricted. The script `netlify:crowdin:downloadTranslationsFailSafe` explicitly states:

```
Crowdin translation download failure (only internal PRs have access to the Crowdin env token)
```

This confirms that **forked pull requests do not have access to the Crowdin environment token**. External contributions cannot read or exfiltration Crowdin secrets through CI.

### CI/CD permissions

The deployment infrastructure supports branch-specific builds:

```json
"netlify:build:production": "pnpm docusaurus write-translations && pnpm netlify:crowdin:delay && pnpm netlify:crowdin:uploadSources && pnpm netlify:crowdin:downloadTranslations && pnpm build",
"netlify:build:branchDeploy": "pnpm build",
"netlify:build:deployPreview": "pnpm build"
```

Production builds execute additional Crowdin integration steps, while branch and preview builds run a standard build without secret access.

## Dependency and supply chain security

### Dependency management

Docusaurus uses **pnpm workspaces** with `workspace:*` protocol for internal packages. Internal packages are linked from the monorepo and are not fetched from the npm registry:

```json
"dependencies": {
  "@docusaurus/core": "workspace:*",
  "@docusaurus/logger": "workspace:*",
  "@docusaurus/plugin-client-redirects": "workspace:*",
  "@docusaurus/plugin-content-blog": "workspace:*",
  "@docusaurus/theme-classic": "workspace:*"
}
```

This reduces supply chain risk for monorepo-internal packages because they are resolved locally rather than from a potentially compromised registry.

### Pinned dependencies

Third-party dependencies are versioned with exact or compatible ranges:

```json
"dependencies": {
  "@crowdin/cli": "^5.0.2",
  "@crowdin/crowdin-api-client": "^1.57.0",
  "react": "^19.3.0",
  "react-dom": "^19.3.0"
}
```

Docusaurus does **not** use a lockfile in the provided manifests. The pnpm lockfile (not included in these source files) governs exact resolution. Pin exact versions for production deployments if reproducible builds are required.

### Known-dependency concerns

The `website` package depends on `@crowdin/cli` and `@crowdin/crowdin-api-client` for translation workflows. These packages handle API credentials. Review all Crowdin API usage in CI scripts before modifying build pipelines.

## Data protection

### Static site output

Docusaurus generates **static HTML, CSS, and JavaScript** files. It does not run server-side code in production. There is no persistent server that stores user data.

### Cookies and tracking

The website includes `@docusaurus/plugin-google-gtag` (Google Analytics) and `@docusaurus/plugin-pwa`. These plugins are declared as dependencies but their configuration is in `docusaurus.config.js`, which is not included in the provided source files. Review that file to confirm:

- Whether Google Analytics is enabled on the documentation site
- Whether PWA service worker caching exposes sensitive data

### Personal data in translations

Crowdin integration stores documentation source strings in the Crowdin cloud service. Do not include secrets, API keys, or personal data in documentation content intended for translation.

## Build and deployment security

### Build process

The build is executed with:

```
pnpm build
```

This runs `docusaurus build`, which uses Webpack 5 (`webpack ^5.111.0`) to bundle assets. Build-time security considerations:

- Docusaurus packages use `webpack-dev-server ^6.0.0` for local development. Never expose the dev server to untrusted networks if sensitive site content is loaded.
- The `serve-handler ^6.1.7` package is used for local static serving. Production deployments should use a configured static file host, not `serve-handler`.

### Visual regression testing

Argos visual diff tests run via Playwright:

```json
"screenshot": "playwright test",
"upload": "pnpm exec -- argos upload ./screenshots/chromium"
```

Ensure that screenshot uploads contain only intended page snapshots. Do not run Argos upload against environments that render secret or user-specific data.

### Base URL configuration

The site supports custom base URLs via environment variable:

```json
"start:baseUrl": "cross-env BASE_URL='/build/' pnpm start",
"build:baseUrl": "cross-env BASE_URL='/build/' pnpm build"
```

Confirm that the deployment base URL is controlled and not user-supplied to prevent path traversal or link injection.

## Security of internal tooling

### Evaluation and bundling tools

The core package depends on `eval ^0.1.8`. This package evaluates JavaScript strings at build time. Its use in Docusaurus is historically for config evaluation, but any future use of `eval` with user-controllable input is a critical security risk.

### LQIP image processing

The `@docusaurus/lqip-loader` package depends on `sharp ^0.35.1`, a native image processing library. Sharp has a history of security patches for image parsing vulnerabilities. Keep `sharp` updated to the latest version and monitor its security advisories.

### Test fixture with outdated dependencies

The `test-bad-package` fixture intentionally uses outdated React versions:

```json
"dependencies": {
  "@mdx-js/react": "1.0.1",
  "react": "16.14.0",
  "react-dom": "16.14.0"
}
```

This package is used to test version conflict handling. It is **not shipped** to end users and does not represent the production dependency posture. Do not copy its dependency versions into production packages.

## Verification limitations

The following security controls are **not verifiable** from the provided package manifests and require additional repository files:

| Control | Requires file not provided |
| --- | --- |
| Content Security Policy headers | `docusaurus.config.js`, hosting config |
| Authentication for admin pages | No auth code exists; Docusaurus is a static generator |
| PWA service worker configuration | `docusaurus.config.js` |
| Google Analytics configuration | `docusaurus.config.js` |
| Dependency vulnerability scanning | CI workflow files (`.github/workflows`) |
| Code signing or artifact integrity | Release configuration |

Review these files in the repository to complete the security assessment.

## Policy enforcement

Docusaurus is a static site generator and does not include runtime authentication or authorization. Security is enforced at the following layers:

1. **Repository access control** — GitHub roles
2. **CI secret scoping** — Crowdin tokens unavailable to fork PRs
3. **Dependency management** — workspace-local package resolution
4. **Static output** — no server-side attack surface in production

By default, Docusaurus sites are public. If you deploy a Docusaurus site with restricted content, configure access control at the hosting provider (for example, Netlify password protection or AWS CloudFront signed URLs), not in the Docusaurus application layer.