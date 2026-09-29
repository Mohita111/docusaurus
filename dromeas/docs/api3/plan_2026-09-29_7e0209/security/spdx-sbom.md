# SPDX SBOM for Docusaurus

## Overview

This document provides a Software Bill of Materials (SBOM) for the Docusaurus repository (Mohita111/docusaurus) in SPDX format. The SBOM covers the packages identified in the repository's `package.json` files, including direct dependencies, devDependencies, and workspace packages.

**SPDX version:** 2.3  
**Data license:** CC0-1.0  
**Document namespace:** `https://github.com/Mohita111/docusaurus/spdx/sbom`

---

## SPDX Document Header

```
SPDXVersion: SPDX-2.3
DataLicense: CC0-1.0
SPDXID: SPDXRef-DOCUMENT
DocumentName: Docusaurus-SBOM
DocumentNamespace: https://github.com/Mohita111/docusaurus/spdx/sbom
Creator: Organization: Docusaurus
Created: 2025-01-01T00:00:00Z
```

---

## Package Inventory

### Top-Level Packages

| Package Name | Version | SPDX Identifier | License | Private |
|---|---|---|---|---|
| `argos` | 4.0.0 | SPDXRef-Package-argos | MIT | Yes |
| `website` | 4.0.0 | SPDXRef-Package-website | Not declared | Yes |
| `@docusaurus/core` | 4.0.0 | SPDXRef-Package-docusaurus-core | MIT | No |
| `@docusaurus/lqip-loader` | 4.0.0 | SPDXRef-Package-lqip-loader | MIT | No |
| `test-bad-package` | 4.0.0 | SPDXRef-Package-test-bad-package | Not declared | Yes |

---

## SPDX Package Definitions

### Package: argos

```
PackageName: argos
SPDXID: SPDXRef-Package-argos
PackageVersion: 4.0.0
PackageLicenseDeclared: MIT
PackageLicenseConcluded: MIT
FilesAnalyzed: true
PackageSupplier: NOASSERTION
PackageDownloadLocation: NOASSERTION

ExternalRef: PACKAGE-MANAGER purl pkg:npm/argos@4.0.0
```

**Dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@argos-ci/cli` | ^6.9.2 | SPDXRef-Package-argos-ci-cli |
| `@argos-ci/playwright` | ^7.5.0 | SPDXRef-Package-argos-ci-playwright |
| `@playwright/test` | ^1.63.0 | SPDXRef-Package-playwright-test |
| `cheerio` | ^1.2.0 | SPDXRef-Package-cheerio |

---

### Package: website

```
PackageName: website
SPDXID: SPDXRef-Package-website
PackageVersion: 4.0.0
PackageLicenseDeclared: NOASSERTION
PackageLicenseConcluded: NOASSERTION
FilesAnalyzed: true
PackageSupplier: NOASSERTION
PackageDownloadLocation: NOASSERTION

ExternalRef: PACKAGE-MANAGER purl pkg:npm/website@4.0.0
```

**Runtime dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@crowdin/cli` | ^5.0.2 | SPDXRef-Package-crowdin-cli |
| `@crowdin/crowdin-api-client` | ^1.57.0 | SPDXRef-Package-crowdin-api-client |
| `@docusaurus/core` | workspace:* | SPDXRef-Package-docusaurus-core |
| `@docusaurus/logger` | workspace:* | SPDXRef-Package-docusaurus-logger |
| `@docusaurus/plugin-client-redirects` | workspace:* | SPDXRef-Package-plugin-client-redirects |
| `@docusaurus/plugin-content-blog` | workspace:* | SPDXRef-Package-plugin-content-blog |
| `@docusaurus/plugin-content-docs` | workspace:* | SPDXRef-Package-plugin-content-docs |
| `@docusaurus/plugin-content-pages` | workspace:* | SPDXRef-Package-plugin-content-pages |
| `@docusaurus/plugin-google-gtag` | workspace:* | SPDXRef-Package-plugin-google-gtag |
| `@docusaurus/plugin-ideal-image` | workspace:* | SPDXRef-Package-plugin-ideal-image |
| `@docusaurus/plugin-pwa` | workspace:* | SPDXRef-Package-plugin-pwa |
| `@docusaurus/plugin-rsdoctor` | workspace:* | SPDXRef-Package-plugin-rsdoctor |
| `@docusaurus/preset-classic` | workspace:* | SPDXRef-Package-preset-classic |
| `@docusaurus/remark-plugin-npm2yarn` | workspace:* | SPDXRef-Package-remark-plugin-npm2yarn |
| `@docusaurus/theme-classic` | workspace:* | SPDXRef-Package-theme-classic |
| `@docusaurus/theme-common` | workspace:* | SPDXRef-Package-theme-common |
| `@docusaurus/theme-live-codeblock` | workspace:* | SPDXRef-Package-theme-live-codeblock |
| `@docusaurus/theme-mermaid` | workspace:* | SPDXRef-Package-theme-mermaid |
| `@docusaurus/types` | workspace:* | SPDXRef-Package-docusaurus-types |
| `@docusaurus/utils` | workspace:* | SPDXRef-Package-docusaurus-utils |
| `@docusaurus/utils-common` | workspace:* | SPDXRef-Package-docusaurus-utils-common |
| `@docusaurus/utils-validation` | workspace:* | SPDXRef-Package-docusaurus-utils-validation |
| `clsx` | ^2.1.1 | SPDXRef-Package-clsx |
| `color` | ^4.2.3 | SPDXRef-Package-color |
| `fs-extra` | ^11.4.0 | SPDXRef-Package-fs-extra |
| `netlify-plugin-cache` | ^1.0.3 | SPDXRef-Package-netlify-plugin-cache |
| `raw-loader` | ^4.0.2 | SPDXRef-Package-raw-loader |
| `react` | ^19.3.0 | SPDXRef-Package-react |
| `react-dom` | ^19.3.0 | SPDXRef-Package-react-dom |
| `react-lite-youtube-embed` | ^3.7.0 | SPDXRef-Package-react-lite-youtube-embed |
| `react-medium-image-zoom` | ^5.4.9 | SPDXRef-Package-react-medium-image-zoom |
| `recma-mdx-displayname` | ^0.4.1 | SPDXRef-Package-recma-mdx-displayname |
| `rehype-katex` | ^7.0.1 | SPDXRef-Package-rehype-katex |
| `remark-math` | ^6.0.0 | SPDXRef-Package-remark-math |
| `unist-util-visit` | ^5.1.0 | SPDXRef-Package-unist-util-visit |
| `webpack` | ^5.111.0 | SPDXRef-Package-webpack |
| `workbox-routing` | ^7.4.1 | SPDXRef-Package-workbox-routing |
| `workbox-strategies` | ^7.4.1 | SPDXRef-Package-workbox-strategies |

**Development dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@docusaurus/eslint-plugin` | workspace:* | SPDXRef-Package-docusaurus-eslint-plugin |
| `@docusaurus/tsconfig` | workspace:* | SPDXRef-Package-docusaurus-tsconfig |
| `@types/color` | ^3.0.7 | SPDXRef-Package-types-color |
| `cross-env` | ^10.1.0 | SPDXRef-Package-cross-env |
| `rimraf` | ^6.1.3 | SPDXRef-Package-rimraf |
| `search-insights` | ^2.17.3 | SPDXRef-Package-search-insights |

---

### Package: @docusaurus/core

```
PackageName: @docusaurus/core
SPDXID: SPDXRef-Package-docusaurus-core
PackageVersion: 4.0.0
PackageLicenseDeclared: MIT
PackageLicenseConcluded: MIT
FilesAnalyzed: true
PackageSupplier: NOASSERTION
PackageDownloadLocation: https://github.com/facebook/docusaurus.git
PackageHomePage: https://github.com/facebook/docusaurus.git

ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/core@4.0.0
```

**Runtime dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@docusaurus/babel` | workspace:* | SPDXRef-Package-docusaurus-babel |
| `@docusaurus/bundler` | workspace:* | SPDXRef-Package-docusaurus-bundler |
| `@docusaurus/logger` | workspace:* | SPDXRef-Package-docusaurus-logger |
| `@docusaurus/mdx-loader` | workspace:* | SPDXRef-Package-docusaurus-mdx-loader |
| `@docusaurus/utils` | workspace:* | SPDXRef-Package-docusaurus-utils |
| `@docusaurus/utils-common` | workspace:* | SPDXRef-Package-docusaurus-utils-common |
| `@docusaurus/utils-validation` | workspace:* | SPDXRef-Package-docusaurus-utils-validation |
| `boxen` | ^6.2.1 | SPDXRef-Package-boxen |
| `chokidar` | ^3.6.0 | SPDXRef-Package-chokidar |
| `cli-table3` | ^0.6.5 | SPDXRef-Package-cli-table3 |
| `combine-promises` | ^1.2.0 | SPDXRef-Package-combine-promises |
| `commander` | ^15.0.0 | SPDXRef-Package-commander |
| `core-js` | ^3.50.0 | SPDXRef-Package-core-js |
| `detect-port` | ^2.1.0 | SPDXRef-Package-detect-port |
| `escape-html` | ^1.0.3 | SPDXRef-Package-escape-html |
| `eta` | ^4.6.0 | SPDXRef-Package-eta |
| `eval` | ^0.1.8 | SPDXRef-Package-eval |
| `execa` | ^10.0.1 | SPDXRef-Package-execa |
| `fs-extra` | ^11.4.0 | SPDXRef-Package-fs-extra |
| `html-tags` | ^5.1.0 | SPDXRef-Package-html-tags |
| `html-webpack-plugin` | ^5.6.8 | SPDXRef-Package-html-webpack-plugin |
| `leven` | ^3.1.0 | SPDXRef-Package-leven |
| `lodash` | ^4.18.1 | SPDXRef-Package-lodash |
| `open` | ^11.0.2 | SPDXRef-Package-open |
| `p-map` | ^4.0.0 | SPDXRef-Package-p-map |
| `prompts` | ^2.4.2 | SPDXRef-Package-prompts |
| `react-helmet-async` | npm:@slorber/react-helmet-async@1.3.0 | SPDXRef-Package-react-helmet-async |
| `react-loadable` | npm:@docusaurus/react-loadable@6.0.0 | SPDXRef-Package-react-loadable |
| `react-loadable-ssr-addon-v5-slorber` | ^1.0.3 | SPDXRef-Package-react-loadable-ssr-addon |
| `react-router` | ^5.3.4 | SPDXRef-Package-react-router |
| `react-router-config` | ^5.1.1 | SPDXRef-Package-react-router-config |
| `react-router-dom` | ^5.3.4 | SPDXRef-Package-react-router-dom |
| `semver` | ^7.8.5 | SPDXRef-Package-semver |
| `serve-handler` | ^6.1.7 | SPDXRef-Package-serve-handler |
| `tinypool` | ^2.1.2 | SPDXRef-Package-tinypool |
| `tslib` | ^2.8.1 | SPDXRef-Package-tslib |
| `update-notifier` | ^6.0.2 | SPDXRef-Package-update-notifier |
| `webpack` | ^5.111.0 | SPDXRef-Package-webpack |
| `webpack-bundle-analyzer` | ^5.3.2 | SPDXRef-Package-webpack-bundle-analyzer |
| `webpack-dev-server` | ^6.0.0 | SPDXRef-Package-webpack-dev-server |
| `webpack-merge` | ^6.0.1 | SPDXRef-Package-webpack-merge |

**Peer dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@docusaurus/faster` | workspace:* | SPDXRef-Package-docusaurus-faster |
| `@mdx-js/react` | ^3.1.1 | SPDXRef-Package-mdx-js-react |
| `react` | ^19.3.0 | SPDXRef-Package-react |
| `react-dom` | ^19.3.0 | SPDXRef-Package-react-dom |

**Development dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@docusaurus/module-type-aliases` | workspace:* | SPDXRef-Package-docusaurus-module-type-aliases |
| `@docusaurus/types` | workspace:* | SPDXRef-Package-docusaurus-types |
| `@total-typescript/shoehorn` | ^0.1.2 | SPDXRef-Package-total-typescript-shoehorn |
| `@types/escape-html` | ^1.0.4 | SPDXRef-Package-types-escape-html |
| `@types/history` | ^4.7.11 | SPDXRef-Package-types-history |
| `@types/react-dom` | ^19.3.0 | SPDXRef-Package-types-react-dom |
| `@types/react-router-config` | ^5.0.11 | SPDXRef-Package-types-react-router-config |
| `@types/react-router-dom` | ^5.3.3 | SPDXRef-Package-types-react-router-dom |
| `@types/serve-handler` | ^6.1.4 | SPDXRef-Package-types-serve-handler |
| `@types/update-notifier` | ^6.0.8 | SPDXRef-Package-types-update-notifier |
| `@types/webpack-bundle-analyzer` | ^4.7.0 | SPDXRef-Package-types-webpack-bundle-analyzer |
| `@types/webpack-env` | ^1.18.8 | SPDXRef-Package-types-webpack-env |
| `tree-node-cli` | ^3.0.0 | SPDXRef-Package-tree-node-cli |

**Environment requirements:**

- Node.js >= 24.21

---

### Package: @docusaurus/lqip-loader

```
PackageName: @docusaurus/lqip-loader
SPDXID: SPDXRef-Package-lqip-loader
PackageVersion: 4.0.0
PackageLicenseDeclared: MIT
PackageLicenseConcluded: MIT
FilesAnalyzed: true
PackageSupplier: NOASSERTION
PackageDownloadLocation: https://github.com/facebook/docusaurus.git

ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/lqip-loader@4.0.0
```

**Runtime dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@docusaurus/logger` | workspace:* | SPDXRef-Package-docusaurus-logger |
| `file-loader` | ^6.2.0 | SPDXRef-Package-file-loader |
| `lodash` | ^4.18.1 | SPDXRef-Package-lodash |
| `sharp` | ^0.35.1 | SPDXRef-Package-sharp |
| `tslib` | ^2.8.1 | SPDXRef-Package-tslib |

**Development dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@types/file-loader` | ^5.0.4 | SPDXRef-Package-types-file-loader |
| `webpack` | ^5.111.0 | SPDXRef-Package-webpack |

**Environment requirements:**

- Node.js >= 24.21

---

### Package: test-bad-package

```
PackageName: test-bad-package
SPDXID: SPDXRef-Package-test-bad-package
PackageVersion: 4.0.0
PackageLicenseDeclared: NOASSERTION
PackageLicenseConcluded: NOASSERTION
FilesAnalyzed: true
PackageSupplier: NOASSERTION
PackageDownloadLocation: NOASSERTION

ExternalRef: PACKAGE-MANAGER purl pkg:npm/test-bad-package@4.0.0
```

**Runtime dependencies:**

| Package Name | Version Constraint | SPDX Identifier |
|---|---|---|
| `@mdx-js/react` | 1.0.1 | SPDXRef-Package-mdx-js-react |
| `react` | 16.14.0 | SPDXRef-Package-react |
| `react-dom` | 16.14.0 | SPDXRef-Package-react-dom |

---

## SPDX Relationships

```
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-argos
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-website
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-lqip-loader
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-test-bad-package

Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-argos-ci-cli
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-argos-ci-playwright
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-playwright-test
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-cheerio

Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-webpack
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-router-dom
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-lodash
Relationship: SPDXRef-Package-lqip-loader DEPENDS_ON SPDXRef-Package-sharp
Relationship: SPDXRef-Package-test-bad-package DEPENDS_ON SPDXRef-Package-mdx-js-react
```

---

## License Summary

| Package | Declared License | SPDX License Expression |
|---|---|---|
| `argos` | MIT | `MIT` |
| `website` | Not declared | `NOASSERTION` |
| `@docusaurus/core` | MIT | `MIT` |
| `@docusaurus/lqip-loader` | MIT | `MIT` |
| `test-bad-package` | Not declared | `NOASSERTION` |

---

## SBOM Generation Guidance

### Using cyclonedx-npm

To generate a CycloneDX-format SBOM from the repository root:

```bash
pnpm dlx --package @cyclonedx/cyclonedx-npm cyclonedx-npm --output-file docusaurus-bom.json
```

### Using syft

To generate an SPDX-format SBOM with syft:

```bash
syft scan dir:. --output spdx-json --file docusaurus.spdx.json
```

### Using SPDX SBOM Generator

To generate an SPDX tag-value SBOM:

```bash
npx --package spdx-sbom-generator spdx-sbom-generator --output . --format tag-value
```

---

## Security Considerations

1. **Workspace dependencies** (`workspace:*`) in the `website` and `@docusaurus/core` packages reference internal packages that must be built from the same repository commit to ensure version consistency.

2. **Version pinning**: Direct dependencies use caret ranges (for example, `^6.9.2`), which allow automatic minor and patch updates. For reproducible security audits, generate a lockfile (`pnpm-lock.yaml`) and use `pnpm install --frozen-lockfile` in CI.

3. **Aliased packages**: The following dependencies use npm aliases that point to non-standard registry locations:
   - `react-helmet-async` → `npm:@slorber/react-helmet-async@1.3.0`
   - `react-loadable` → `npm:@docusaurus/react-loadable@6.0.0`

4. **Node.js runtime**: `@docusaurus/core` and `@docusaurus/lqip-loader` require Node.js >= 24.21. Verify the runtime environment meets this requirement.

5. **Private packages**: `argos`, `website`, and `test-bad-package` are marked as `private: true`, meaning they are not published to the npm registry. SBOM consumers should treat them as internal-only components.

6. **Peer dependencies**: `@docusaurus/core` lists `@docusaurus/faster` as an optional peer dependency. SBOM tooling should account for optional peer relationships to avoid false-positive vulnerability matches.

---

## SBOM Data Limitations

The following information is not available from the provided `package.json` files and should be supplemented from lockfile analysis:

- Exact resolved versions for caret-ranged dependencies
- Transitive dependencies (dependencies of dependencies)
- Package verification codes and file-level hashes
- Copyright notices and supplier information for third-party packages
- License information for packages without `license` fields (`website` and `test-bad-package`)

To produce a complete SBOM, run one of the generation tools listed in the SBOM Generation Guidance section against the full repository with a resolved lockfile.