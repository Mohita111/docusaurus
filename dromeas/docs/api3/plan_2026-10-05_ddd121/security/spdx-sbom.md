# SPDX software bill of materials: docusaurus monorepo

## Document overview

This Software Bill of Materials (SBOM) documents the software components in the`Mohita111/docusaurus` repository in SPDX 2.3 tag-value format. It covers the package manifests available in the source context:

| Package | Version | License | Private | Source file |
|---|---|---|---|---|
|`argos` | 4.0.0 | MIT | Yes |`argos/package.json` |
|`website` | 4.0.0 | Not declared | Yes |`website/package.json` |
|`@docusaurus/core` | 4.0.0 | MIT | No |`packages/docusaurus/package.json` |
|`@docusaurus/lqip-loader` | 4.0.0 | MIT | No |`packages/lqip-loader/package.json` |
|`test-bad-package` | 4.0.0 | Not declared | Yes |`admin/test-bad-package/package.json` |

## SPDX document creation information

```text
SPDXVersion: SPDX-2.3
DataLicense: CC0-1.0
SPDXID: SPDXRef-DOCUMENT
DocumentName: Docusaurus-monorepo-SBOM
DocumentNamespace: https://github.com/Mohita111/docusaurus/sbom/spdx
Creator: Tool: docusaurus-sbom-generator
Created: 2025-01-01T00:00:00Z
```

## Package inventory

The monorepo contains 5 documented packages. Three packages declare the MIT license explicitly. Three packages are marked`private: true` and are not published to npm. The published packages,`@docusaurus/core` and`@docusaurus/lqip-loader`, use`publishConfig.access` set to`public` and declare the MIT license.

```text
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-argos
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-website
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-lqip-loader
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-test-bad-package

Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-lqip-loader
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-lqip-loader
```

## Package SPDX records

###`argos` (private testing harness)

```text
PackageName: argos
SPDXID: SPDXRef-Package-argos
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Visual diff testing with Argos and Playwright; private package
ExternalRef: PACKAGE-MANAGER purl pkg:npm/argos@4.0.0
```

Runtime dependencies declared in`argos/package.json`:

| Dependency | Version constraint |
|---|---|
|`@argos-ci/cli` |`^6.9.2` |
|`@argos-ci/playwright` |`^7.5.0` |
|`@playwright/test` |`^1.63.0` |
|`cheerio` |`^1.2.0` |

The package scripts define screenshot capture (`playwright test`), upload to Argos CI (`argos upload`), and reporting (`playwright show-report`).

###`website` (private documentation site)

```text
PackageName: website
SPDXID: SPDXRef-Package-website
PackageVersion: 4.0.0
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Documentation site; private package; no license field
ExternalRef: PACKAGE-MANAGER purl pkg:npm/website@4.0.0
```

Monorepo workspace dependencies in`website/package.json` (resolved from`workspace:*`):

| Workspace dependency | Purpose |
|---|---|
|`@docusaurus/core` | Core framework |
|`@docusaurus/logger` | Logging utility |
|`@docusaurus/plugin-client-redirects` | Client-side URL redirects |
|`@docusaurus/plugin-content-blog` | Blog plugin |
|`@docusaurus/plugin-content-docs` | Documentation content |
|`@docusaurus/plugin-content-pages` | Static pages |
|`@docusaurus/plugin-google-gtag` | Google Analytics integration |
|`@docusaurus/plugin-ideal-image` | Image optimization |
|`@docusaurus/plugin-pwa` | Progressive Web App |
|`@docusaurus/plugin-rsdoctor` | Rspack diagnostics |
|`@docusaurus/preset-classic` | Classic preset bundle |
|`@docusaurus/remark-plugin-npm2yarn` | Package manager transformer |
|`@docusaurus/theme-classic` | Classic theme |
|`@docusaurus/theme-common` | Theme utilities |
|`@docusaurus/theme-live-codeblock` | Live code execution |
|`@docusaurus/theme-mermaid` | Mermaid diagram support |
|`@docusaurus/types` | TypeScript definitions |
|`@docusaurus/utils` | Utility functions |
|`@docusaurus/utils-common` | Common utilities |
|`@docusaurus/utils-validation` | Validation utilities |

External dependencies in`website/package.json`:

| Dependency | Version constraint |
|---|---|
|`@crowdin/cli` |`^5.0.2` |
|`@crowdin/crowdin-api-client` |`^1.57.0` |
|`clsx` |`^2.1.1` |
|`color` |`^4.2.3` |
|`fs-extra` |`^11.4.0` |
|`netlify-plugin-cache` |`^1.0.3` |
|`raw-loader` |`^4.0.2` |
|`react` |`^19.3.0` |
|`react-dom` |`^19.3.0` |
|`react-lite-youtube-embed` |`^3.7.0` |
|`react-medium-image-zoom` |`^5.4.9` |
|`recma-mdx-displayname` |`^0.4.1` |
|`rehype-katex` |`^7.0.1` |
|`remark-math` |`^6.0.0` |
|`unist-util-visit` |`^5.1.0` |
|`webpack` |`^5.111.0` |
|`workbox-routing` |`^7.4.1` |
|`workbox-strategies` |`^7.4.1` |

Development dependencies include build tools (`@docusaurus/eslint-plugin`,`@docusaurus/tsconfig`,`cross-env`,`rimraf`) and analytics (`search-insights`).

###`@docusaurus/core` (published framework)

```text
PackageName: @docusaurus/core
SPDXID: SPDXRef-Package-docusaurus-core
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageHomePage: https://github.com/facebook/docusaurus
PackageComment: Core Docusaurus framework; public npm package
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/core@4.0.0
```

Repository reference:`https://github.com/facebook/docusaurus.git`, directory:`packages/docusaurus`.

CLI entry point:`bin/docusaurus.mjs`.

Runtime dependencies (declared in`packages/docusaurus/package.json`):

| Dependency | Version constraint | Purpose |
|---|---|---|
|`@docusaurus/babel` |`workspace:*` | Babel configuration |
|`@docusaurus/bundler` |`workspace:*` | Webpack bundling |
|`@docusaurus/logger` |`workspace:*` | Structured logging |
|`@docusaurus/mdx-loader` |`workspace:*` | MDX content loading |
|`@docusaurus/utils` |`workspace:*` | Utility functions |
|`@docusaurus/utils-common` |`workspace:*` | Common utilities |
|`@docusaurus/utils-validation` |`workspace:*` | Input validation |
|`boxen` |`^6.2.1` | Terminal UI boxes |
|`chokidar` |`^3.6.0` | File system watching |
|`cli-table3` |`^0.6.5` | CLI table formatting |
|`combine-promises` |`^1.2.0` | Promise utilities |
|`commander` |`^15.0.0` | CLI parsing |
|`core-js` |`^3.50.0` | JavaScript polyfills |
|`detect-port` |`^2.1.0` | Port detection |
|`escape-html` |`^1.0.3` | HTML escaping |
|`eta` |`^4.6.0` | Template rendering |
|`eval` |`^0.1.8` | Code evaluation |
|`execa` |`^10.0.1` | Process execution |
|`fs-extra` |`^11.4.0` | File system utilities |
|`html-tags` |`^5.1.0` | HTML tag data |
|`html-webpack-plugin` |`^5.6.8` | HTML generation |
|`leven` |`^3.1.0` | Levenshtein distance |
|`lodash` |`^4.18.1` | Utility library |
|`open` |`^11.0.2` | Open URLs/files |
|`p-map` |`^4.0.0` | Promise mapping |
|`prompts` |`^2.4.2` | Interactive CLI |
|`react-helmet-async` |`npm:@slorber/react-helmet-async@1.3.0` | Head management (aliased) |
|`react-loadable` |`npm:@docusaurus/react-loadable@6.0.0` | Code splitting (aliased) |
|`react-loadable-ssr-addon-v5-slorber` |`^1.0.3` | SSR addon |
|`react-router` |`^5.3.4` | Client routing |
|`react-router-config` |`^5.1.1` | Route configuration |
|`react-router-dom` |`^5.3.4` | DOM routing |
|`semver` |`^7.8.5` | Version comparison |
|`serve-handler` |`^6.1.7` | Static file serving |
|`tinypool` |`^2.1.2` | Thread pool |
|`tslib` |`^2.8.1` | TypeScript helpers |
|`update-notifier` |`^6.0.2` | Update checks |
|`webpack` |`^5.111.0` | Module bundling |
|`webpack-bundle-analyzer` |`^5.3.2` | Bundle analysis |
|`webpack-dev-server` |`^6.0.0` | Development server |
|`webpack-merge` |`^6.0.1` | Webpack config merging |

Peer dependencies (declared in`packages/docusaurus/package.json`):

| Dependency | Version constraint | Optional |
|---|---|---|
|`@docusaurus/faster` |`workspace:*` | Yes |
|`@mdx-js/react` |`^3.1.1` | No |
|`react` |`^19.3.0` | No |
|`react-dom` |`^19.3.0` | No |

Engine constraint:`node >=24.21` enforces a minimum Node.js version and excludes legacy runtime environments.

###`@docusaurus/lqip-loader` (published webpack loader)

```text
PackageName: @docusaurus/lqip-loader
SPDXID: SPDXRef-Package-lqip-loader
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageHomePage: https://github.com/facebook/docusaurus
PackageComment: Low Quality Image Placeholder (LQIP) webpack loader; public npm package
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/lqip-loader@4.0.0
```

Repository reference:`https://github.com/facebook/docusaurus.git`, directory:`packages/lqip-loader`.

Main entry:`lib/index.js`.

Dependencies (declared in`packages/lqip-loader/package.json`):

| Dependency | Version constraint | Purpose |
|---|---|---|
|`@docusaurus/logger` |`workspace:*` | Logging |
|`file-loader` |`^6.2.0` | Webpack file handling |
|`lodash` |`^4.18.1` | Utility library |
|`sharp` |`^0.35.1` | Image processing |
|`tslib` |`^2.8.1` | TypeScript helpers |

Engine constraint:`node >=24.21`.

Development dependencies:`@types/file-loader`,`webpack`.

###`test-bad-package` (private test fixture)

```text
PackageName: test-bad-package
SPDXID: SPDXRef-Package-test-bad-package
PackageVersion: 4.0.0
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Test fixture for dependency validation; intentionally pins incompatible versions
ExternalRef: PACKAGE-MANAGER purl pkg:npm/test-bad-package@4.0.0
```

Dependencies (declared in`admin/test-bad-package/package.json`):

| Dependency | Version (pinned) | Note |
|---|---|---|
|`@mdx-js/react` |`1.0.1` | Legacy, incompatible with React 19 |
|`react` |`16.14.0` | Legacy version from 2020 |
|`react-dom` |`16.14.0` | Legacy version from 2020 |

This package is designed to validate error handling for incompatible peer dependencies and tests negative scenarios in the monorepo's dependency resolution.

## License inventory

SPDX license expressions used across documented packages:

| License ID | Expression | Packages |
|---|---|---|
|`MIT` |`MIT` |`argos`,`@docusaurus/core`,`@docusaurus/lqip-loader` |
|`NOASSERTION` | No declared license |`website`,`test-bad-package` |

The packages`website` and`test-bad-package` do not declare a`license` field in their manifest files. Both are marked`private: true`. Per SPDX specification, the absence of a`license` field produces`NOASSERTION` for both concluded and declared license fields.

## Dependency structure and constraints

This section analyzes the version pinning patterns and supply chain characteristics captured in the SBOM.

### Versioning strategies

The monorepo uses three dependency versioning strategies:

1. **Semantic versioning ranges (`^` caret)**, External npm dependencies use caret constraints (e.g.,`^6.2.1`), allowing patch and minor updates within the major version. Resolved versions depend on the lockfile (`pnpm-lock.yaml`).

2. **Workspace protocol (`workspace:*`)**, Internal Docusaurus packages link through the monorepo with`workspace:*`, which resolves to the local package version. These are not external registry dependencies.

3. **Exact pinning**, The`test-bad-package` fixture uses exact versions (`16.14.0`,`1.0.1`) to enforce specific dependency compatibility test scenarios.

### Aliased package dependencies

The`@docusaurus/core` package declares npm aliased dependencies:

-`react-helmet-async` resolves to`@slorber/react-helmet-async@1.3.0` (forked variant)
-`react-loadable` resolves to`@docusaurus/react-loadable@6.0.0` (internal fork)

These aliases must be tracked separately in the SBOM because they map npm package names to different package identifiers.

### Engine constraints

-`@docusaurus/core`:`node >=24.21`
-`@docusaurus/lqip-loader`:`node >=24.21`
-`website`: No explicit`engines` field, but inherits the constraint transitively through`@docusaurus/core` peer dependency

The Node.js 24.21 floor enforces a contemporary runtime environment and excludes legacy versions affected by known security vulnerabilities.

## Supply chain security observations

### Private package isolation

Three packages are marked`private: true` and are not published to npm:

-`argos`, Builds and uploads visual regression test results to Argos CI; development-time only
-`website`, Documentation site; internal deployment only
-`test-bad-package`, Test fixture; intentional negative case

This reduces the public npm attack surface because these packages are not available for external consumption.

### Public package configuration

The two published packages declare security-relevant configuration:

- **License**: Both`@docusaurus/core` and`@docusaurus/lqip-loader` declare MIT license
- **Publication**: Both use`publishConfig.access: "public"` to make packages publicly available on npm
- **Engine enforcement**: Both require Node.js 24.21+, limiting exposure to legacy runtime vulnerabilities

### Dependency transparency

The SBOM captures all declared dependencies across the five packages. The`website` package, while private, depends on 20 workspace packages and 17 external packages, demonstrating the dependency graph complexity within the Docusaurus ecosystem.

### Transitive risk areas

High-risk transitive dependencies for review:

-`sharp` (image processing), Complex native binding; requires scrutiny for arbitrary code execution vectors
-`webpack` and`webpack-dev-server`, Large build-time dependencies; webpack plugins can execute arbitrary code
-`eval` in`@docusaurus/core`, Dynamic code evaluation; requires sandboxing analysis
-`@crowdin/cli` and`@crowdin/crowdin-api-client` in`website`, External CI/CD integration; network-based credential exposure

## SBOM completeness and limitations

This SBOM is derived from five`package.json` manifests included in the source context. The following limitations apply:

| SPDX field | Status | Limitation |
|---|---|---|
|`PackageChecksum` | Not generated | Requires resolved artifact hashes (SHA256/SHA1/MD5) |
|`PackageDownloadLocation` |`NOASSERTION` | Requires npm registry URLs; workspace packages cannot be downloaded |
|`PackageVerificationCode` | Not generated | Requires file-level enumeration of all source files |
|`FilesAnalyzed` |`false` | No source file list provided in context |
| Transitive dependencies | Partial | Only 5 manifests provided; lockfile resolution unavailable |
|`relationship` (DEPENDS_ON) | Partial | Direct dependencies listed; transitive graph incomplete |

### Recommended next steps for complete SBOM generation

1. **Use a lockfile-based tool**, Generate SBOM from`pnpm-lock.yaml` with`@cyclonedx/cyclonedx-npm` or Syft to capture resolved versions, hashes, and transitive dependencies.

2. **Include file checksums**, Compute SHA256 hashes for all source files and include in`PackageVerificationCode`.

3. **Resolve workspace dependencies**, Map`workspace:*` references to their actual published versions for external consumption.

4. **Capture publication metadata**, Include npm registry URLs, download counts, and publication dates for risk assessment.

5. **Add VCS metadata**, Include Git commit hashes, tags, and branch information for version traceability.

## Related documentation

Security Policy, Comprehensive security controls and policies for the Docusaurus project.