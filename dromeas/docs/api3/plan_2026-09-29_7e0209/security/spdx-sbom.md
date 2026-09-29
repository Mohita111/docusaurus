# SPDX Software Bill of Materials

## Overview

This document provides a Software Bill of Materials (SBOM) for the `Mohita111/docusaurus` repository in SPDX tag-value format (SPDX 2.3). The SBOM inventories the packages declared in the repository's `package.json` manifests and their respective dependency graphs.

The SBOM is scoped to the five package manifests provided in the repository source:

- `argos/package.json`
- `website/package.json`
- `packages/docusaurus/package.json`
- `packages/lqip-loader/package.json`
- `admin/test-bad-package/package.json`

## How to use this SBOM

- Use the `PackageName` and `PackageVersion` fields to identify each component.
- Use `Relationship` entries to trace the dependency graph from the root document to individual packages.
- Use `ExternalRef: SECURITY` and `ExternalRef: PACKAGE-MANAGER` entries to correlate components with npm registry data and vulnerability databases.
- Cross-reference `PackageLicenseConcluded` values with your organization's license compliance policy.

## SBOM in SPDX tag-value format

```spdx
SPDXVersion: SPDX-2.3
DataLicense: CC0-1.0
SPDXID: SPDXRef-DOCUMENT
DocumentName: Mohita111-docusaurus-sbom
DocumentNamespace: https://spdx.org/spdxdocs/docusaurus/4.0.0/mohita111-docusaurus
Creator: Tool: manually-generated-docs
Created: 2026-01-01T00:00:00Z

##### Root repository

PackageName: docusaurus-monorepo
SPDXID: SPDXRef-Package-docusaurus-monorepo
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
PackageCopyrightText: NOASSERTION
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-docusaurus-monorepo

##### Package: argos

PackageName: argos
SPDXID: SPDXRef-Package-argos
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/argos@4.0.0
Relationship: SPDXRef-Package-docusaurus-monorepo CONTAINS SPDXRef-Package-argos
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-argos-cli
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-argos-playwright
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-playwright-test
Relationship: SPDXRef-Package-argos DEPENDS_ON SPDXRef-Package-cheerio

##### Package: website

PackageName: website
SPDXID: SPDXRef-Package-website
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/website@4.0.0
Relationship: SPDXRef-Package-docusaurus-monorepo CONTAINS SPDXRef-Package-website
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-crowdin-cli
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-crowdin-api-client
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-clsx
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-color
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-fs-extra
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-netlify-plugin-cache
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-raw-loader
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-react
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-react-dom
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-react-lite-youtube-embed
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-react-medium-image-zoom
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-recma-mdx-displayname
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-rehype-katex
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-remark-math
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-unist-util-visit
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-webpack
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-workbox-routing
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-workbox-strategies
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-docusaurus-eslint-plugin
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-docusaurus-tsconfig
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-types-color
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-cross-env
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-rimraf
Relationship: SPDXRef-Package-website DEV_DEPENDENCY_OF SPDXRef-Package-search-insights

##### Package: @docusaurus/core

PackageName: @docusaurus/core
SPDXID: SPDXRef-Package-docusaurus-core
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/core@4.0.0
Relationship: SPDXRef-Package-docusaurus-monorepo CONTAINS SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-boxen
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-chokidar
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-cli-table3
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-combine-promises
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-commander
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-core-js
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-detect-port
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-escape-html
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-eta
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-eval
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-execa
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-fs-extra
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-html-tags
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-html-webpack-plugin
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-leven
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-lodash
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-open
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-p-map
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-prompts
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-helmet-async
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-loadable
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-loadable-ssr-addon-v5-slorber
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-router
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-router-config
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-react-router-dom
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-semver
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-serve-handler
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-tinypool
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-tslib
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-update-notifier
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-webpack
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-webpack-bundle-analyzer
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-webpack-dev-server
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-webpack-merge

##### Package: @docusaurus/lqip-loader

PackageName: @docusaurus/lqip-loader
SPDXID: SPDXRef-Package-docusaurus-lqip-loader
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/lqip-loader@4.0.0
Relationship: SPDXRef-Package-docusaurus-monorepo CONTAINS SPDXRef-Package-docusaurus-lqip-loader
Relationship: SPDXRef-Package-docusaurus-lqip-loader DEPENDS_ON SPDXRef-Package-file-loader
Relationship: SPDXRef-Package-docusaurus-lqip-loader DEPENDS_ON SPDXRef-Package-lodash
Relationship: SPDXRef-Package-docusaurus-lqip-loader DEPENDS_ON SPDXRef-Package-sharp
Relationship: SPDXRef-Package-docusaurus-lqip-loader DEPENDS_ON SPDXRef-Package-tslib
Relationship: SPDXRef-Package-docusaurus-lqip-loader DEPENDS_ON SPDXRef-Package-docusaurus-logger

##### Package: test-bad-package

PackageName: test-bad-package
SPDXID: SPDXRef-Package-test-bad-package
PackageVersion: 4.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/test-bad-package@4.0.0
Relationship: SPDXRef-Package-docusaurus-monorepo CONTAINS SPDXRef-Package-test-bad-package
Relationship: SPDXRef-Package-test-bad-package DEPENDS_ON SPDXRef-Package-mdx-js-react
Relationship: SPDXRef-Package-test-bad-package DEPENDS_ON SPDXRef-Package-react-16
Relationship: SPDXRef-Package-test-bad-package DEPENDS_ON SPDXRef-Package-react-dom-16

##### External dependencies — argos

PackageName: @argos-ci/cli
SPDXID: SPDXRef-Package-argos-cli
PackageVersion: 6.9.2
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40argos-ci/cli@6.9.2

PackageName: @argos-ci/playwright
SPDXID: SPDXRef-Package-argos-playwright
PackageVersion: 7.5.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40argos-ci/playwright@7.5.0

PackageName: @playwright/test
SPDXID: SPDXRef-Package-playwright-test
PackageVersion: 1.63.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40playwright/test@1.63.0

PackageName: cheerio
SPDXID: SPDXRef-Package-cheerio
PackageVersion: 1.2.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/cheerio@1.2.0

##### External dependencies — website

PackageName: @crowdin/cli
SPDXID: SPDXRef-Package-crowdin-cli
PackageVersion: 5.0.2
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40crowdin/cli@5.0.2

PackageName: @crowdin/crowdin-api-client
SPDXID: SPDXRef-Package-crowdin-api-client
PackageVersion: 1.57.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40crowdin/crowdin-api-client@1.57.0

PackageName: clsx
SPDXID: SPDXRef-Package-clsx
PackageVersion: 2.1.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/clsx@2.1.1

PackageName: color
SPDXID: SPDXRef-Package-color
PackageVersion: 4.2.3
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/color@4.2.3

PackageName: fs-extra
SPDXID: SPDXRef-Package-fs-extra
PackageVersion: 11.4.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/fs-extra@11.4.0

PackageName: netlify-plugin-cache
SPDXID: SPDXRef-Package-netlify-plugin-cache
PackageVersion: 1.0.3
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/netlify-plugin-cache@1.0.3

PackageName: raw-loader
SPDXID: SPDXRef-Package-raw-loader
PackageVersion: 4.0.2
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/raw-loader@4.0.2

PackageName: react
SPDXID: SPDXRef-Package-react
PackageVersion: 19.3.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/react@19.3.0

PackageName: react-dom
SPDXID: SPDXRef-Package-react-dom
PackageVersion: 19.3.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/react-dom@19.3.0

PackageName: react-lite-youtube-embed
SPDXID: SPDXRef-Package-react-lite-youtube-embed
PackageVersion: 3.7.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/react-lite-youtube-embed@3.7.0

PackageName: react-medium-image-zoom
SPDXID: SPDXRef-Package-react-medium-image-zoom
PackageVersion: 5.4.9
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/react-medium-image-zoom@5.4.9

PackageName: recma-mdx-displayname
SPDXID: SPDXRef-Package-recma-mdx-displayname
PackageVersion: 0.4.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/recma-mdx-displayname@0.4.1

PackageName: rehype-katex
SPDXID: SPDXRef-Package-rehype-katex
PackageVersion: 7.0.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/rehype-katex@7.0.1

PackageName: remark-math
SPDXID: SPDXRef-Package-remark-math
PackageVersion: 6.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/remark-math@6.0.0

PackageName: unist-util-visit
SPDXID: SPDXRef-Package-unist-util-visit
PackageVersion: 5.1.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/unist-util-visit@5.1.0

PackageName: webpack
SPDXID: SPDXRef-Package-webpack
PackageVersion: 5.111.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/webpack@5.111.0

PackageName: workbox-routing
SPDXID: SPDXRef-Package-workbox-routing
PackageVersion: 7.4.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/workbox-routing@7.4.1

PackageName: workbox-strategies
SPDXID: SPDXRef-Package-workbox-strategies
PackageVersion: 7.4.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/workbox-strategies@7.4.1

##### External devDependencies — website

PackageName: @types/color
SPDXID: SPDXRef-Package-types-color
PackageVersion: 3.0.7
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40types/color@3.0.7

PackageName: cross-env
SPDXID: SPDXRef-Package-cross-env
PackageVersion: 10.1.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/cross-env@10.1.0

PackageName: rimraf
SPDXID: SPDXRef-Package-rimraf
PackageVersion: 6.1.3
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/rimraf@6.1.3

PackageName: search-insights
SPDXID: SPDXRef-Package-search-insights
PackageVersion: 2.17.3
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/search-insights@2.17.3

##### External dependencies — @docusaurus/core

PackageName: boxen
SPDXID: SPDXRef-Package-boxen
PackageVersion: 6.2.1
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/boxen@6.2.1

PackageName: chokidar
SPDXID: SPDXRef-Package-chokidar
PackageVersion: 3.6.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/chokidar@3.6.0

PackageName: cli-table3
SPDXID: SPDXRef-Package-cli-table3
PackageVersion: 0.6.5
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/cli-table3@0.6.5

PackageName: combine-promises
SPDXID: SPDXRef-Package-combine-promises
PackageVersion: 1.2.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/combine-promises@1.2.0

PackageName: commander
SPDXID: SPDXRef-Package-commander
PackageVersion: 15.0.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/commander@15.0.0

PackageName: core-js
SPDXID: SPDXRef-Package-core-js
PackageVersion: 3.50.0
PackageDownloadLocation: NOASSERTION
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
ExternalRef: PACKAGE-MANAGER purl pkg:npm/core-js@3.50.0

PackageName: detect-port
SPDXID