# Security Policy

## Purpose

This document describes security-relevant policies and access-control metadata that can be verified from the provided Docusaurus repository manifest files. The source context includes five `package.json` files:

- `argos/package.json`
- `website/package.json`
- `packages/docusaurus/package.json`
- `packages/lqip-loader/package.json`
- `admin/test-bad-package/package.json`

The provided files do not include `SECURITY.md`, lockfiles, CI workflows, server configuration, or application source code. Therefore, this document covers manifest-level controls only. Runtime authentication, authorization, encryption, and vulnerability remediation are not represented.

## Publishing and dependency visibility

The `private` and `publishConfig` fields control whether a package can be published to the npm registry and who can access it.

| Manifest file | Package name | `private` | `publishConfig` | `license` |
|---|---|---|---|---|
| `argos/package.json` | `argos` | `true` | not set | `MIT` |
| `website/package.json` | `website` | `true` | not set | not set |
| `packages/docusaurus/package.json` | `@docusaurus/core` | not set | `"access": "public"` | `MIT` |
| `packages/lqip-loader/package.json` | `@docusaurus/lqip-loader` | not set | `"access": "public"` | `MIT` |
| `admin/test-bad-package/package.json` | `test-bad-package` | `true` | not set | not set |

The private packages are excluded from registry publication. The two publishable Docusaurus packages have explicit public access on npm.

## Runtime engine policy

The published packages declare a minimum Node.js version through the `engines` field.

- `packages/docusaurus/package.json`:
  ```json
  "engines": {
    "node": ">=24.21"
  }
  ```

- `packages/lqip-loader/package.json`:
  ```json
  "engines": {
    "node": ">=24.21"
  }
  ```

The `website` package does not declare an `engines` field in the provided manifest.

## Access-controlled deployment workflows

`website/package.json` contains Netlify scripts that reference Crowdin translation upload and download. The `netlify:crowdin:downloadTranslationsFailSafe` script includes an access-control fallback:

```json
"netlify:crowdin:downloadTranslationsFailSafe": "pnpm netlify:crowdin:wait && (pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')"
```

This script documents that the Crowdin environment token is available only to internal pull requests. External pull request builds fall back to a non-fatal message instead of receiving the token.

Related workflows in `website/package.json`:

```json
"netlify:crowdin:downloadTranslations": "pnpm netlify:crowdin:wait && pnpm --dir .. crowdin:download:website"
"netlify:crowdin:uploadSources": "pnpm --dir .. crowdin:upload:website"
```

Both scripts invoke Crowdin commands from the repository root when the Crowdin token is accessible.

## Dependency surface

The manifests declare runtime dependencies that may be relevant for supply chain reviews.

`packages/docusaurus/package.json` includes:

- `webpack`
- `webpack-dev-server`
- `serve-handler`
- `chokidar`
- `commander`
- `detect-port`
- `execa`
- `react-router`
- `react-helmet-async`

`packages/lqip-loader/package.json` includes:

- `sharp`
- `file-loader`
- `lodash`

`argos/package.json` includes:

- `@argos-ci/cli`
- `@argos-ci/playwright`
- `@playwright/test`
- `cheerio`

These lists are direct dependencies only. Transitive dependencies and lockfile pins are not present in the provided source context.

## SBOM and vulnerability disclosure

No SBOM configuration is declared in the provided manifests. The dependency metadata supports generation of CycloneDX or SPDX documents, but no SBOM generation command or format is specified.

The only public contact metadata in the manifests is:

- Repository: `https://github.com/facebook/docusaurus.git`
- Bugs URL: `https://github.com/facebook/docusaurus/issues`

A dedicated vulnerability disclosure channel is not declared in the provided files. A `SECURITY.md` file was not part of the source context, so no official reporting policy can be documented from this data.