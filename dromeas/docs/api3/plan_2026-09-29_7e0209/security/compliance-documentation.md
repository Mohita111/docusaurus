# Compliance Documentation: Docusaurus Repository

This document provides regulatory compliance and audit information for the `Mohita111/docusaurus` repository based on the five package manifests supplied for analysis: `argos/package.json`, `website/package.json`, `packages/docusaurus/package.json`, `packages/lqip-loader/package.json`, and `admin/test-bad-package/package.json`.

## Scope

This compliance documentation covers:

- License declarations present in package manifests
- Dependency inventory for the supplied package files
- Node.js runtime requirements
- Security-relevant scripts and environment handling
- Software Bill of Materials (SBOM) format requirements
- Audit-relevant test and verification controls

Only the package manifests listed above were available. Full CI workflow files, lockfiles, and source-level configuration are outside the evidence set.

## License compliance

The repository uses the MIT license for public packages where a license is declared.

| Package manifest | License field |
| --- | --- |
| `packages/docusaurus/package.json` | `"license": "MIT"` |
| `packages/lqip-loader/package.json` | `"license": "MIT"` |
| `argos/package.json` | `"license": "MIT"` |
| `website/package.json` | No `license` field present |
| `admin/test-bad-package/package.json` | No `license` field present |

`website/package.json` and `admin/test-bad-package/package.json` are marked `"private": true`. Their lack of a `license` field does not indicate a missing license at the repository level, but the absence of an SPDX identifier in those manifests should be noted for audit completeness.

## Dependency inventory

The supplied manifests define the top-level dependency surface.

### `packages/docusaurus/package.json`

Scoped package: `@docusaurus/core` version `4.0.0`.

Security-relevant dependencies include:

| Dependency | Declared version |
| --- | --- |
| `webpack` | `^5.111.0` |
| `webpack-dev-server` | `^6.0.0` |
| `serve-handler` | `^6.1.7` |
| `escape-html` | `^1.0.3` |
| `eval` | `^0.1.8` |
| `react-router` | `^5.3.4` |
| `react-router-dom` | `^5.3.4` |
| `react-router-config` | `^5.1.1` |

Peer dependency requirements:

| Peer dependency | Required version |
| --- | --- |
| `react` | `^19.3.0` |
| `react-dom` | `^19.3.0` |
| `@docusaurus/faster` | `workspace:*` optional |
| `@mdx-js/react` | `^3.1.1` |

Runtime engine:

| Engine | Requirement |
| --- | --- |
| `node` | `>=24.21` |

The presence of `eval` as a dependency should be reviewed during code audit to determine whether it processes untrusted input. The supplied manifests do not include call sites.

### `packages/lqip-loader/package.json`

Scoped package: `@docusaurus/lqip-loader` version `4.0.0`.

Security-relevant dependencies include:

| Dependency | Declared version |
| --- | --- |
| `sharp` | `^0.35.1` |
| `file-loader` | `^6.2.0` |
| `lodash` | `^4.18.1` |

Runtime engine:

| Engine | Requirement |
| --- | --- |
| `node` | `>=24.21` |

### `argos/package.json`

Package: `argos` version `4.0.0`.

Dependencies used for visual regression testing:

| Dependency | Declared version |
| --- | --- |
| `@argos-ci/cli` | `^6.9.2` |
| `@argos-ci/playwright` | `^7.5.0` |
| `@playwright/test` | `^1.63.0` |
| `cheerio` | `^1.2.0` |

### `website/package.json`

Package: `website` version `4.0.0`, private.

Security-relevant plugins and build dependencies include:

| Dependency | Declared version |
| --- | --- |
| `@docusaurus/plugin-pwa` | `workspace:*` |
| `@docusaurus/plugin-google-gtag` | `workspace:*` |
| `@docusaurus/theme-live-codeblock` | `workspace:*` |
| `@docusaurus/theme-mermaid` | `workspace:*` |
| `@crowdin/cli` | `^5.0.2` |
| `@crowdin/crowdin-api-client` | `^1.57.0` |
| `webpack` | `^5.111.0` |
| `workbox-routing` | `^7.4.1` |
| `workbox-strategies` | `^7.4.1` |
| `react` | `^19.3.0` |
| `react-dom` | `^19.3.0` |

The Google gtag plugin and PWA plugin can collect analytics or register service workers. Their runtime configuration is not included in the supplied package manifest.

### `admin/test-bad-package/package.json`

Package: `test-bad-package` version `4.0.0`, private.

This manifest deliberately pins legacy versions:

| Dependency | Declared version |
| --- | --- |
| `@mdx-js/react` | `1.0.1` |
| `react` | `16.14.0` |
| `react-dom` | `16.14.0` |

This package appears intended for negative testing or compatibility coverage rather than production deployment. Auditors should confirm that no production build uses these versions.

## Node.js runtime requirements

Node `>=24.21` is required by both `packages/docusaurus/package.json` and `packages/lqip-loader/package.json`.

No `engines` field is present in `website/package.json`, `argos/package.json`, or `admin/test-bad-package/package.json`.

## Software Bill of Materials requirements

The provided package manifests do not contain dedicated `sbom`, `packageManager`, or SPDX document fields. To satisfy SBOM requirements, generate one of the following from the repository lockfile:

- SPDX 2.3 or later
- CycloneDX 1.5 or later

The supplied files demonstrate pnpm usage through scripts such as:

```json
"upload": "pnpm exec -- argos upload ./screenshots/chromium"
```

References:

- `argos/package.json`
- `website/package.json`

No SBOM generation script is declared in the supplied manifests.

## Audit controls

The repository includes automated verification scripts that support audit and compliance review.

### Visual regression testing

`argos/package.json` defines:

```json
"screenshot": "playwright test",
"upload": "pnpm exec -- argos upload ./screenshots/chromium",
"upload-text-snapshots": "pnpm exec -- argos upload \"../website/build\" --build-name text-snapshots --files styles.css --files \"docs/**/*.html\" --files \"blog/**/*.html\"",
"report": "playwright show-report"
```

These scripts are relevant to compliance for documenting rendered output and detecting unintended visual changes.

### Type checking

`website/package.json` defines:

```json
"typecheck": "tsc"
```

### Swizzle theme tests

`website/package.json` includes four swizzle-related tests that verify theme customization against JS and TypeScript variations:

- `test:swizzle:eject:js`
- `test:swizzle:eject:ts`
- `test:swizzle:wrap:js`
- `test:swizzle:wrap:ts`

These controls help ensure that theme modifications do not introduce regressions.

### CSS order validation

`website/package.json` includes:

```json
"test:css-order": "node testCSSOrder.mjs"
```

This provides deterministic verification of stylesheet ordering.

## Sensitive environment and credential handling

`website/package.json` contains Crowdin translation workflows that require a Crowdin token. The relevant scripts are:

```json
"netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website",
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')",
"netlify:crowdin:uploadSources": "pnpm --dir .. crowdin:upload:website"
```

The failsafe script explicitly states that only internal PRs can access the Crowdin environment token. Auditors should confirm that no Crowdin credentials are stored in the repository or in public CI logs. The exact environment variable name is not present in the supplied manifest.

## Compliance limitations

The evidence set consists only of package manifests. The following items are not verifiable from the supplied files:

- Full dependency tree and transitive dependency licenses
- Webpack production security configuration
- PWA service worker runtime settings
- Google gtag data collection configuration
- Source-level usage of the `eval` dependency in `@docusaurus/core`
- CI workflow access controls beyond the Crowdin fail-safe comment

A complete compliance assessment requires the repository lockfile, CI configuration, and source files for the packages listed above.