# CycloneDX SBOM

## Overview

CycloneDX is a lightweight software bill of materials (SBOM) standard designed for use in application security contexts and supply chain component analysis. This document describes the CycloneDX SBOM for the Docusaurus monorepo (repository`Mohita111/docusaurus`) and the tooling to generate and maintain it.

The CycloneDX SBOM covers all package manifests across the monorepo:

| Manifest path | Package name | Version | Scope |
|---------------|--------------|---------|-------|
|`argos/package.json` |`argos` | 4.0.0 | Visual diff test tooling |
|`website/package.json` |`website` | 4.0.0 | Documentation site |
|`packages/docusaurus/package.json` |`@docusaurus/core` | 4.0.0 | Core framework |
|`packages/lqip-loader/package.json` |`@docusaurus/lqip-loader` | 4.0.0 | Image placeholder loader |
|`admin/test-bad-package/package.json` |`test-bad-package` | 4.0.0 | Test fixture for dependency validation |

All packages use`"private": true` or publish through the`@docusaurus` npm scope. The monorepo uses pnpm workspaces, denoted by the`workspace:*` protocol in dependencies.

## CycloneDX document structure

The SBOM root document uses CycloneDX 1.5 JSON format with the following structure:

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "serialNumber": "urn:uuid:3e671687-395b-41f5-a30f-58985a69b264",
  "version": 1,
  "metadata": {
    "timestamp": "2025-07-14T00:00:00Z",
    "component": {
      "bom-ref": "pkg:npm/%40docusaurus/monorepo@4.0.0",
      "type": "application",
      "name": "docusaurus-monorepo",
      "version": "4.0.0"
    },
    "tools": [
      {
        "vendor": "OWASP",
        "name": "cdxgen",
        "version": "10.0.0"
      }
    ]
  },
  "components": []
}
```

## Component inventory

### Application components

These components are direct entries from the package manifests:

| Component |`bom-ref` | PURL | Version |
|-----------|-----------|------|---------|
|`argos` |`pkg:npm/argos@4.0.0` |`pkg:npm/argos@4.0.0` | 4.0.0 |
|`website` |`pkg:npm/website@4.0.0` |`pkg:npm/website@4.0.0` | 4.0.0 |
|`@docusaurus/core` |`pkg:npm/%40docusaurus/core@4.0.0` |`pkg:npm/%40docusaurus/core@4.0.0` | 4.0.0 |
|`@docusaurus/lqip-loader` |`pkg:npm/%40docusaurus/lqip-loader@4.0.0` |`pkg:npm/%40docusaurus/lqip-loader@4.0.0` | 4.0.0 |
|`test-bad-package` |`pkg:npm/test-bad-package@4.0.0` |`pkg:npm/test-bad-package@4.0.0` | 4.0.0 |

###`argos` dependencies

The`argos` package at`argos/package.json` declares four runtime dependencies for visual regression testing:

| Component | Version | PURL | Type |
|-----------|---------|------|------|
|`@argos-ci/cli` | >=6.9.2 <7.0.0 |`pkg:npm/%40argos-ci/cli@6.9.2` | dependency |
|`@argos-ci/playwright` | >=7.5.0 <8.0.0 |`pkg:npm/%40argos-ci/playwright@7.5.0` | dependency |
|`@playwright/test` | >=1.63.0 <2.0.0 |`pkg:npm/%40playwright/test@1.63.0` | dependency |
|`cheerio` | >=1.2.0 <2.0.0 |`pkg:npm/cheerio@1.2.0` | dependency |

All`argos` dependencies are runtime dependencies used for visual regression testing via Playwright. The`@argos-ci/cli` component connects to the Argos CI service to upload screenshots; SBOM consumers should note the supply chain risk from this third-party CI integration.

###`website` production dependencies

The`website` package at`website/package.json` declares 38 production dependencies. Workspace references (`workspace:*`) resolve to version 4.0.0 at publish time:

| Component | Version | PURL |
|-----------|---------|------|
|`@crowdin/cli` | >=5.0.2 <6.0.0 |`pkg:npm/%40crowdin/cli@5.0.2` |
|`@crowdin/crowdin-api-client` | >=1.57.0 <2.0.0 |`pkg:npm/%40crowdin/crowdin-api-client@1.57.0` |
|`@docusaurus/core` |`workspace:*` |`pkg:npm/%40docusaurus/core@4.0.0` |
|`@docusaurus/logger` |`workspace:*` |`pkg:npm/%40docusaurus/logger@4.0.0` |
|`@docusaurus/plugin-client-redirects` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-client-redirects@4.0.0` |
|`@docusaurus/plugin-content-blog` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-content-blog@4.0.0` |
|`@docusaurus/plugin-content-docs` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-content-docs@4.0.0` |
|`@docusaurus/plugin-content-pages` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-content-pages@4.0.0` |
|`@docusaurus/plugin-google-gtag` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-google-gtag@4.0.0` |
|`@docusaurus/plugin-ideal-image` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-ideal-image@4.0.0` |
|`@docusaurus/plugin-pwa` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-pwa@4.0.0` |
|`@docusaurus/plugin-rsdoctor` |`workspace:*` |`pkg:npm/%40docusaurus/plugin-rsdoctor@4.0.0` |
|`@docusaurus/preset-classic` |`workspace:*` |`pkg:npm/%40docusaurus/preset-classic@4.0.0` |
|`@docusaurus/remark-plugin-npm2yarn` |`workspace:*` |`pkg:npm/%40docusaurus/remark-plugin-npm2yarn@4.0.0` |
|`@docusaurus/theme-classic` |`workspace:*` |`pkg:npm/%40docusaurus/theme-classic@4.0.0` |
|`@docusaurus/theme-common` |`workspace:*` |`pkg:npm/%40docusaurus/theme-common@4.0.0` |
|`@docusaurus/theme-live-codeblock` |`workspace:*` |`pkg:npm/%40docusaurus/theme-live-codeblock@4.0.0` |
|`@docusaurus/theme-mermaid` |`workspace:*` |`pkg:npm/%40docusaurus/theme-mermaid@4.0.0` |
|`@docusaurus/types` |`workspace:*` |`pkg:npm/%40docusaurus/types@4.0.0` |
|`@docusaurus/utils` |`workspace:*` |`pkg:npm/%40docusaurus/utils@4.0.0` |
|`@docusaurus/utils-common` |`workspace:*` |`pkg:npm/%40docusaurus/utils-common@4.0.0` |
|`@docusaurus/utils-validation` |`workspace:*` |`pkg:npm/%40docusaurus/utils-validation@4.0.0` |
|`clsx` | >=2.1.1 <3.0.0 |`pkg:npm/clsx@2.1.1` |
|`color` | >=4.2.3 <5.0.0 |`pkg:npm/color@4.2.3` |
|`fs-extra` | >=11.4.0 <12.0.0 |`pkg:npm/fs-extra@11.4.0` |
|`netlify-plugin-cache` | >=1.0.3 <2.0.0 |`pkg:npm/netlify-plugin-cache@1.0.3` |
|`raw-loader` | >=4.0.2 <5.0.0 |`pkg:npm/raw-loader@4.0.2` |
|`react` | >=19.3.0 <20.0.0 |`pkg:npm/react@19.3.0` |
|`react-dom` | >=19.3.0 <20.0.0 |`pkg:npm/react-dom@19.3.0` |
|`react-lite-youtube-embed` | >=3.7.0 <4.0.0 |`pkg:npm/react-lite-youtube-embed@3.7.0` |
|`react-medium-image-zoom` | >=5.4.9 <6.0.0 |`pkg:npm/react-medium-image-zoom@5.4.9` |
|`recma-mdx-displayname` | >=0.4.1 <0.5.0 |`pkg:npm/recma-mdx-displayname@0.4.1` |
|`rehype-katex` | >=7.0.1 <8.0.0 |`pkg:npm/rehype-katex@7.0.1` |
|`remark-math` | >=6.0.0 <7.0.0 |`pkg:npm/remark-math@6.0.0` |
|`unist-util-visit` | >=5.1.0 <6.0.0 |`pkg:npm/unist-util-visit@5.1.0` |
|`webpack` | >=5.111.0 <6.0.0 |`pkg:npm/webpack@5.111.0` |
|`workbox-routing` | >=7.4.1 <8.0.0 |`pkg:npm/workbox-routing@7.4.1` |
|`workbox-strategies` | >=7.4.1 <8.0.0 |`pkg:npm/workbox-strategies@7.4.1` |

###`website` development dependencies

The`website` package declares 6 development dependencies:

| Component | Version | PURL |
|-----------|---------|------|
|`@docusaurus/eslint-plugin` |`workspace:*` |`pkg:npm/%40docusaurus/eslint-plugin@4.0.0` |
|`@docusaurus/tsconfig` |`workspace:*` |`pkg:npm/%40docusaurus/tsconfig@4.0.0` |
|`@types/color` | >=3.0.7 <4.0.0 |`pkg:npm/%40types/color@3.0.7` |
|`cross-env` | >=10.1.0 <11.0.0 |`pkg:npm/cross-env@10.1.0` |
|`rimraf` | >=6.1.3 <7.0.0 |`pkg:npm/rimraf@6.1.3` |
|`search-insights` | >=2.17.3 <3.0.0 |`pkg:npm/search-insights@2.17.3` |

###`@docusaurus/core` dependencies

The`@docusaurus/core` package at`packages/docusaurus/package.json` declares 42 production dependencies and 8 development dependencies. Key production dependencies include:

| Component | Version | PURL |
|-----------|---------|------|
|`@docusaurus/babel` |`workspace:*` |`pkg:npm/%40docusaurus/babel@4.0.0` |
|`@docusaurus/bundler` |`workspace:*` |`pkg:npm/%40docusaurus/bundler@4.0.0` |
|`@docusaurus/logger` |`workspace:*` |`pkg:npm/%40docusaurus/logger@4.0.0` |
|`@docusaurus/mdx-loader` |`workspace:*` |`pkg:npm/%40docusaurus/mdx-loader@4.0.0` |
|`@docusaurus/utils` |`workspace:*` |`pkg:npm/%40docusaurus/utils@4.0.0` |
|`@docusaurus/utils-common` |`workspace:*` |`pkg:npm/%40docusaurus/utils-common@4.0.0` |
|`@docusaurus/utils-validation` |`workspace:*` |`pkg:npm/%40docusaurus/utils-validation@4.0.0` |
|`boxen` | >=6.2.1 <7.0.0 |`pkg:npm/boxen@6.2.1` |
|`chokidar` | >=3.6.0 <4.0.0 |`pkg:npm/chokidar@3.6.0` |
|`cli-table3` | >=0.6.5 <0.7.0 |`pkg:npm/cli-table3@0.6.5` |
|`combine-promises` | >=1.2.0 <2.0.0 |`pkg:npm/combine-promises@1.2.0` |
|`commander` | >=15.0.0 <16.0.0 |`pkg:npm/commander@15.0.0` |
|`core-js` | >=3.50.0 <4.0.0 |`pkg:npm/core-js@3.50.0` |
|`detect-port` | >=2.1.0 <3.0.0 |`pkg:npm/detect-port@2.1.0` |
|`escape-html` | >=1.0.3 <2.0.0 |`pkg:npm/escape-html@1.0.3` |
|`eta` | >=4.6.0 <5.0.0 |`pkg:npm/eta@4.6.0` |
|`eval` | >=0.1.8 <0.2.0 |`pkg:npm/eval@0.1.8` |
|`execa` | >=10.0.1 <11.0.0 |`pkg:npm/execa@10.0.1` |
|`fs-extra` | >=11.4.0 <12.0.0 |`pkg:npm/fs-extra@11.4.0` |
|`html-tags` | >=5.1.0 <6.0.0 |`pkg:npm/html-tags@5.1.0` |
|`html-webpack-plugin` | >=5.6.8 <6.0.0 |`pkg:npm/html-webpack-plugin@5.6.8` |
|`leven` | >=3.1.0 <4.0.0 |`pkg:npm/leven@3.1.0` |
|`lodash` | >=4.18.1 <5.0.0 |`pkg:npm/lodash@4.18.1` |
|`open` | >=11.0.2 <12.0.0 |`pkg:npm/open@11.0.2` |
|`p-map` | >=4.0.0 <5.0.0 |`pkg:npm/p-map@4.0.0` |
|`prompts` | >=2.4.2 <3.0.0 |`pkg:npm/prompts@2.4.2` |
|`react-helmet-async` | ^1.3.0 (aliased) |`pkg:npm/@slorber/react-helmet-async@1.3.0` |
|`react-loadable` | ^6.0.0 (aliased) |`pkg:npm/@docusaurus/react-loadable@6.0.0` |
|`react-loadable-ssr-addon-v5-slorber` | >=1.0.3 <2.0.0 |`pkg:npm/react-loadable-ssr-addon-v5-slorber@1.0.3` |
|`react-router` | >=5.3.4 <6.0.0 |`pkg:npm/react-router@5.3.4` |
|`react-router-config` | >=5.1.1 <6.0.0 |`pkg:npm/react-router-config@5.1.1` |
|`react-router-dom` | >=5.3.4 <6.0.0 |`pkg:npm/react-router-dom@5.3.4` |
|`semver` | >=7.8.5 <8.0.0 |`pkg:npm/semver@7.8.5` |
|`serve-handler` | >=6.1.7 <7.0.0 |`pkg:npm/serve-handler@6.1.7` |
|`tinypool` | >=2.1.2 <3.0.0 |`pkg:npm/tinypool@2.1.2` |
|`tslib` | >=2.8.1 <3.0.0 |`pkg:npm/tslib@2.8.1` |
|`update-notifier` | >=6.0.2 <7.0.0 |`pkg:npm/update-notifier@6.0.2` |
|`webpack` | >=5.111.0 <6.0.0 |`pkg:npm/webpack@5.111.0` |
|`webpack-bundle-analyzer` | >=5.3.2 <6.0.0 |`pkg:npm/webpack-bundle-analyzer@5.3.2` |
|`webpack-dev-server` | >=6.0.0 <7.0.0 |`pkg:npm/webpack-dev-server@6.0.0` |
|`webpack-merge` | >=6.0.1 <7.0.0 |`pkg:npm/webpack-merge@6.0.1` |

The package specifies`"engines": {"node": ">=24.21"}`, establishing the minimum runtime requirement for supported deployment.

###`@docusaurus/lqip-loader` dependencies

The`@docusaurus/lqip-loader` package at`packages/lqip-loader/package.json` declares 4 production dependencies:

| Component | Version | PURL |
|-----------|---------|------|
|`@docusaurus/logger` |`workspace:*` |`pkg:npm/%40docusaurus/logger@4.0.0` |
|`file-loader` | >=6.2.0 <7.0.0 |`pkg:npm/file-loader@6.2.0` |
|`lodash` | >=4.18.1 <5.0.0 |`pkg:npm/lodash@4.18.1` |
|`sharp` | >=0.35.1 <0.36.0 |`pkg:npm/sharp@0.35.1` |
|`tslib` | >=2.8.1 <3.0.0 |`pkg:npm/tslib@2.8.1` |

The package requires Node.js 24.21 or later via`"engines": {"node": ">=24.21"}`. The`sharp` library is a native image processing dependency that requires system libraries; deployments should ensure native build tools are available.

###`test-bad-package` dependencies

The`test-bad-package` fixture at`admin/test-bad-package/package.json` intentionally pins outdated versions:

| Component | Version | PURL | Status |
|-----------|---------|------|--------|
|`@mdx-js/react` | 1.0.1 (pinned) |`pkg:npm/%40mdx-js/react@1.0.1` | Obsolete major version |
|`react` | 16.14.0 (pinned) |`pkg:npm/react@16.14.0` | End of life |
|`react-dom` | 16.14.0 (pinned) |`pkg:npm/react-dom@16.14.0` | End of life |

This fixture exists to validate that the monorepo's dependency checks detect mismatched peer dependencies. SBOM consumers must exclude this package from production risk assessments and supply chain analysis.

## Dependencies graph

CycloneDX represents relationships between components with the`dependencies` array at the root level. Each entry specifies a reference component and the components it depends upon.

### Workspace dependency relationships

The monorepo uses`workspace:*` to reference sibling packages at the same version. Each workspace reference resolves to version 4.0.0 at publish time. The core dependency relationships are:

```
website → @docusaurus/core
website → @docusaurus/plugin-client-redirects
website → @docusaurus/plugin-content-blog
website → @docusaurus/plugin-content-docs
website → @docusaurus/plugin-content-pages
website → @docusaurus/plugin-google-gtag
website → @docusaurus/plugin-ideal-image
website → @docusaurus/plugin-pwa
website → @docusaurus/plugin-rsdoctor
website → @docusaurus/preset-classic
website → @docusaurus/theme-classic
website → @docusaurus/theme-common
website → @docusaurus/theme-live-codeblock
website → @docusaurus/theme-mermaid
website → @docusaurus/types
website → @docusaurus/utils
website → @docusaurus/utils-common
website → @docusaurus/utils-validation
website → @docusaurus/remark-plugin-npm2yarn
website → @docusaurus/logger

@docusaurus/core → @docusaurus/babel
@docusaurus/core → @docusaurus/bundler
@docusaurus/core → @docusaurus/logger
@docusaurus/core → @docusaurus/mdx-loader
@docusaurus/core → @docusaurus/utils
@docusaurus/core → @docusaurus/utils-common
@docusaurus/core → @docusaurus/utils-validation

@docusaurus/lqip-loader → @docusaurus/logger
```

### Example CycloneDX dependencies entry

The`dependencies` array in a CycloneDX SBOM uses this structure:

```json
{
  "dependencies": [
    {
      "ref": "pkg:npm/website@4.0.0",
      "dependsOn": [
        "pkg:npm/%40docusaurus/core@4.0.0",
        "pkg:npm/%40docusaurus/plugin-client-redirects@4.0.0"
      ]
    },
    {
      "ref": "pkg:npm/%40docusaurus/core@4.0.0",
      "dependsOn": [
        "pkg:npm/%40docusaurus/babel@4.0.0",
        "pkg:npm/%40docusaurus/bundler@4.0.0",
        "pkg:npm/%40docusaurus/logger@4.0.0",
        "pkg:npm/webpack@5.111.0"
      ]
    }
  ]
}
```

## Generating the SBOM

The CycloneDX SBOM is generated from the pnpm lockfile using tools like`cdxgen` or`@cyclonedx/npm`. The repository root contains a pnpm workspace configuration, so the lockfile is the authoritative source of resolved versions.

### Prerequisites

The monorepo requires Node.js 24.21 or later, as declared in`packages/docusaurus/package.json`:

```json
"engines": {
  "node": ">=24.21"
}
```

Ensure your runtime meets this requirement before running generation tools.

### Generation with cdxgen

From the repository root, generate the CycloneDX SBOM using the cdxgen tool:

```bash
npx @cyclonedx/cdxgen -t npm -o bom.json
```

The`cdxgen` tool resolves workspace dependencies from the pnpm lockfile and emits a CycloneDX 1.5 JSON document containing all transitive dependencies. Use the`-t npm` flag to specify the npm package manager type.

### Generation with cyclonedx-npm

Alternatively, use the official CycloneDX npm package:

```bash
npx @cyclonedx/cyclonedx-npm --output-file bom.json --output-format json
```

### Validation

After generation, validate the SBOM against the CycloneDX 1.5 JSON schema:

```bash
npx @cyclonedx/cyclonedx-cli validate --input-file bom.json --input-format json --schema-version 1.5
```

This verification step ensures the document conforms to the CycloneDX specification and can be reliably consumed by downstream tools.

## Security-relevant SBOM fields

### License declarations

All primary application packages declare the MIT license in their`package.json` files:

| Package | License field | Value |
|---------|---------------|-------|
|`argos` |`"license"` |`"MIT"` |
|`website` | (omitted) | Not explicitly declared |
|`@docusaurus/core` |`"license"` |`"MIT"` |
|`@docusaurus/lqip-loader` |`"license"` |`"MIT"` |
|`test-bad-package` | (omitted) | Not explicitly declared |

The`website` and`test-bad-package` packages omit the`"license"` field from their`package.json` manifests. CycloneDX generation tools may emit`"licenses": []` for these components. If exact SPDX license tracking is required for compliance, add explicit license declarations to these manifests.

### Aliased and forked dependencies

Two dependencies in`@docusaurus/core` use npm aliases to reference security-patched forks:

| Manifest name | Resolved package | Version | Purpose |
|---------------|------------------|---------|---------|
|`react-helmet-async` |`@slorber/react-helmet-async` | 1.3.0 | Fork with security fixes for async rendering |
|`react-loadable` |`@docusaurus/react-loadable` | 6.0.0 | Fork with security fixes for code splitting |

CycloneDX must record these aliases as separate components with their resolved package names, not the manifest aliases. Use`externalReferences` with type`distribution` to document the package origin:

```json
{
  "bom-ref": "pkg:npm/%40slorber/react-helmet-async@1.3.0",
  "type": "library",
  "name": "@slorber/react-helmet-async",
  "version": "1.3.0",
  "purl": "pkg:npm/%40slorber/react-helmet-async@1.3.0",
  "externalReferences": [
    {
      "type": "distribution",
      "url": "https://www.npmjs.com/package/@slorber/react-helmet-async"
    }
  ]
}
```

### End-of-life and deprecated components

The`test-bad-package` fixture uses end-of-life versions. These dependencies must be excluded from production SBOM generation:

| Component | Version | EOL status | Exclusion reason |
|-----------|---------|-----------|------------------|
|`react` | 16.14.0 | End of life; no upstream support | Test fixture only; never used in production |
|`react-dom` | 16.14.0 | End of life; no upstream support | Test fixture only; never used in production |
|`@mdx-js/react` | 1.0.1 | Obsolete major version; v3 current | Test fixture only; never used in production |

When generating production SBOMs, exclude the`test-bad-package` component:

```bash
npx @cyclonedx/cdxgen -t npm -o bom.json --exclude-packages test-bad-package
```

### Production versus development scope

CycloneDX supports the`scope` field on components to classify production and development dependencies. For this monorepo:

| Package | Production components | Development components | Total |
|---------|----------------------|------------------------|-------|
|`website` | 38 | 6 | 44 |
|`@docusaurus/core` | 42 | 8 | 50 |
|`@docusaurus/lqip-loader` | 5 | 1 | 6 |
|`argos` | 4 | 0 | 4 |
|`test-bad-package` | 3 | 0 | 3 |

When generating production release SBOMs, mark development-only components with`"scope": "excluded"`:

```json
{
  "type": "library",
  "bom-ref": "pkg:npm/cross-env@10.1.0",
  "name": "cross-env",
  "version": "10.1.0",
  "purl": "pkg:npm/cross-env@10.1.0",
  "scope": "excluded"
}
```

## Example full CycloneDX JSON for a single package

The following fragment illustrates the complete component encoding for`@docusaurus/core` with a subset of its dependencies:

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "serialNumber": "urn:uuid:3e671687-395b-41f5-a30f-58985a69b264",
  "version": 1,
  "metadata": {
    "timestamp": "2025-07-14T00:00:00Z",
    "component": {
      "type": "application",
      "bom-ref": "pkg:npm/%40docusaurus/core@4.0.0",
      "name": "@docusaurus/core",
      "version": "4.0.0",
      "licenses": [{"license": {"id": "MIT"}}]
    }
  },
  "components": [
    {
      "type": "library",
      "bom-ref": "pkg:npm/webpack@5.111.0",
      "name": "webpack",
      "version": "5.111.0",
      "purl": "pkg:npm/webpack@5.111.0",
      "scope": "required"
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/lodash@4.18.1",
      "name": "lodash",
      "version":