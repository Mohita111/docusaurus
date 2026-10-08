# SPDX Software Bill of Materials — Docusaurus Monorepo

## Document overview

This Software Bill of Materials (SBOM) documents the software components in the `Mohita111/docusaurus` repository in SPDX 2.3 tag-value format. It covers the package manifests available in the pre-fetched source context:

| Package | Version | License | Private | Source file |
|---|---|---|---|---|
| `argos` | 4.0.0 | MIT | Yes | `argos/package.json` |
| `website` | 4.0.0 | Not declared | Yes | `website/package.json` |
| `@docusaurus/core` | 4.0.0 | MIT | No | `packages/docusaurus/package.json` |
| `@docusaurus/lqip-loader` | 4.0.0 | MIT | No | `packages/lqip-loader/package.json` |
| `test-bad-package` | 4.0.0 | Not declared | Yes | `admin/test-bad-package/package.json` |

## SPDX document creation information

```text
SPDXVersion: SPDX-2.3
DataLicense: CC0-1.0
SPDXID: SPDXRef-DOCUMENT
DocumentName: Docusaurus-monorepo-SBOM
DocumentNamespace: https://github.com/Mohita111/docusaurus/sbom/spdx
Creator: Tool: docusaurus-sbom-generator
Created: 2026-01-01T00:00:00Z
```

## Package inventory

The monorepo contains 5 documented packages. Two packages declare the MIT license explicitly. Three packages are marked `private: true` and are not published to npm. The published packages, `@docusaurus/core` and `@docusaurus/lqip-loader`, use `publishConfig.access` set to `public` and declare the MIT license.

```text
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-argos
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-website
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-lqip-loader
Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-test-bad-package

Relationship: SPDXRef-Package-website DEPENDS_ON SPDXRef-Package-docusaurus-core
Relationship: SPDXRef-Package-docusaurus-core DEPENDS_ON SPDXRef-Package-lqip-loader
```

## Package SPDX records

### `argos` (private)

```text
PackageName: argos
SPDXID: SPDXRef-Package-argos
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Visual diff testing harness; private package, not published
ExternalRef: PACKAGE-MANAGER purl pkg:npm/argos@4.0.0
```

Dependencies in `argos/package.json`:

| Dependency | Version range |
|---|---|
| `@argos-ci/cli` | `^6.9.2` |
| `@argos-ci/playwright` | `^7.5.0` |
| `@playwright/test` | `^1.63.0` |
| `cheerio` | `^1.2.0` |

### `website` (private)

```text
PackageName: website
SPDXID: SPDXRef-Package-website
PackageVersion: 4.0.0
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Documentation site; private package; no license field declared
ExternalRef: PACKAGE-MANAGER purl pkg:npm/website@4.0.0
```

Workspace dependencies in `website/package.json`:

| Dependency | Version |
|---|---|
| `@docusaurus/core` | `workspace:*` |
| `@docusaurus/logger` | `workspace:*` |
| `@docusaurus/plugin-client-redirects` | `workspace:*` |
| `@docusaurus/plugin-content-blog` | `workspace:*` |
| `@docusaurus/plugin-content-docs` | `workspace:*` |
| `@docusaurus/plugin-content-pages` | `workspace:*` |
| `@docusaurus/plugin-google-gtag` | `workspace:*` |
| `@docusaurus/plugin-ideal-image` | `workspace:*` |
| `@docusaurus/plugin-pwa` | `workspace:*` |
| `@docusaurus/plugin-rsdoctor` | `workspace:*` |
| `@docusaurus/preset-classic` | `workspace:*` |
| `@docusaurus/remark-plugin-npm2yarn` | `workspace:*` |
| `@docusaurus/theme-classic` | `workspace:*` |
| `@docusaurus/theme-common` | `workspace:*` |
| `@docusaurus/theme-live-codeblock` | `workspace:*` |
| `@docusaurus/theme-mermaid` | `workspace:*` |
| `@docusaurus/types` | `workspace:*` |
| `@docusaurus/utils` | `workspace:*` |
| `@docusaurus/utils-common` | `workspace:*` |
| `@docusaurus/utils-validation` | `workspace:*` |

External `website` dependencies:

| Dependency | Version range |
|---|---|
| `@crowdin/cli` | `^5.0.2` |
| `@crowdin/crowdin-api-client` | `^1.57.0` |
| `clsx` | `^2.1.1` |
| `color` | `^4.2.3` |
| `fs-extra` | `^11.4.0` |
| `netlify-plugin-cache` | `^1.0.3` |
| `raw-loader` | `^4.0.2` |
| `react` | `^19.3.0` |
| `react-dom` | `^19.3.0` |
| `react-lite-youtube-embed` | `^3.7.0` |
| `react-medium-image-zoom` | `^5.4.9` |
| `recma-mdx-displayname` | `^0.4.1` |
| `rehype-katex` | `^7.0.1` |
| `remark-math` | `^6.0.0` |
| `unist-util-visit` | `^5.1.0` |
| `webpack` | `^5.111.0` |
| `workbox-routing` | `^7.4.1` |
| `workbox-strategies` | `^7.4.1` |

### `@docusaurus/core` (published)

```text
PackageName: @docusaurus/core
SPDXID: SPDXRef-Package-docusaurus-core
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageHomePage: https://github.com/facebook/docusaurus.git
PackageComment: Core Docusaurus framework; public npm package
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/core@4.0.0
```

The `repository` field points to `https://github.com/facebook/docusaurus.git` with `directory: packages/docusaurus`. The `bin` entry exposes the CLI at `bin/docusaurus.mjs`.

Runtime dependencies:

| Dependency | Version range |
|---|---|
| `@docusaurus/babel` | `workspace:*` |
| `@docusaurus/bundler` | `workspace:*` |
| `@docusaurus/logger` | `workspace:*` |
| `@docusaurus/mdx-loader` | `workspace:*` |
| `@docusaurus/utils` | `workspace:*` |
| `@docusaurus/utils-common` | `workspace:*` |
| `@docusaurus/utils-validation` | `workspace:*` |
| `boxen` | `^6.2.1` |
| `chokidar` | `^3.6.0` |
| `cli-table3` | `^0.6.5` |
| `combine-promises` | `^1.2.0` |
| `commander` | `^15.0.0` |
| `core-js` | `^3.50.0` |
| `detect-port` | `^2.1.0` |
| `escape-html` | `^1.0.3` |
| `eta` | `^4.6.0` |
| `eval` | `^0.1.8` |
| `execa` | `^10.0.1` |
| `fs-extra` | `^11.4.0` |
| `html-tags` | `^5.1.0` |
| `html-webpack-plugin` | `^5.6.8` |
| `leven` | `^3.1.0` |
| `lodash` | `^4.18.1` |
| `open` | `^11.0.2` |
| `p-map` | `^4.0.0` |
| `prompts` | `^2.4.2` |
| `react-helmet-async` | `npm:@slorber/react-helmet-async@1.3.0` |
| `react-loadable` | `npm:@docusaurus/react-loadable@6.0.0` |
| `react-loadable-ssr-addon-v5-slorber` | `^1.0.3` |
| `react-router` | `^5.3.4` |
| `react-router-config` | `^5.1.1` |
| `react-router-dom` | `^5.3.4` |
| `semver` | `^7.8.5` |
| `serve-handler` | `^6.1.7` |
| `tinypool` | `^2.1.2` |
| `tslib` | `^2.8.1` |
| `update-notifier` | `^6.0.2` |
| `webpack` | `^5.111.0` |
| `webpack-bundle-analyzer` | `^5.3.2` |
| `webpack-dev-server` | `^6.0.0` |
| `webpack-merge` | `^6.0.1` |

Peer dependencies:

| Dependency | Version range | Optional |
|---|---|---|
| `@docusaurus/faster` | `workspace:*` | Yes |
| `@mdx-js/react` | `^3.1.1` | No |
| `react` | `^19.3.0` | No |
| `react-dom` | `^19.3.0` | No |

The Node.js engine is `>=24.21`.

### `@docusaurus/lqip-loader` (published)

```text
PackageName: @docusaurus/lqip-loader
SPDXID: SPDXRef-Package-lqip-loader
PackageVersion: 4.0.0
PackageLicenseConcluded: MIT
PackageLicenseDeclared: MIT
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageHomePage: https://github.com/facebook/docusaurus.git
PackageComment: LQIP webpack loader; public npm package
ExternalRef: PACKAGE-MANAGER purl pkg:npm/%40docusaurus/lqip-loader@4.0.0
```

Dependencies:

| Dependency | Version range |
|---|---|
| `@docusaurus/logger` | `workspace:*` |
| `file-loader` | `^6.2.0` |
| `lodash` | `^4.18.1` |
| `sharp` | `^0.35.1` |
| `tslib` | `^2.8.1` |

The Node.js engine is `>=24.21`.

### `test-bad-package` (private)

```text
PackageName: test-bad-package
SPDXID: SPDXRef-Package-test-bad-package
PackageVersion: 4.0.0
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
FilesAnalyzed: false
PackageDownloadLocation: NOASSERTION
PackageComment: Fixture package for dependency validation tests; private; intentionally pins incompatible React versions
ExternalRef: PACKAGE-MANAGER purl pkg:npm/test-bad-package@4.0.0
```

Dependencies:

| Dependency | Version (pinned) |
|---|---|
| `@mdx-js/react` | `1.0.1` |
| `react` | `16.14.0` |
| `react-dom` | `16.14.0` |

## License inventory

SPDX license expressions used across the documented packages:

| License ID | Expression | Packages |
|---|---|---|
| `MIT` | `MIT` | `argos`, `@docusaurus/core`, `@docusaurus/lqip-loader` |
| `NOASSERTION` | Not declared | `website`, `test-bad-package` |

The root monorepo packages `website` and `test-bad-package` do not declare a `license` field in their `package.json` files. Both are marked `private: true`. SPDX requires either a concluded or declared license; the absence of a `license` field produces `NOASSERTION`.

## Version pinning and supply chain observations

The SBOM captures the following versioning patterns:

1. **Semver ranges (`^`)** — Most external dependencies use caret ranges, meaning the resolved versions may change within the major version. For reproducible SBOMs, lockfile resolution is required.
2. **Workspace references (`workspace:*`)** — Internal Docusaurus packages are linked through the monorepo workspace. The SBOM cannot resolve these to concrete immutable versions without the lockfile.
3. **Pinned versions (`test-bad-package`)** — The fixture package uses exact versions (`16.14.0`, `1.0.1`) to test dependency compatibility failures.
4. **npm aliased packages** — `@docusaurus/core` declares two aliased dependencies: `react-helmet-async` resolves to `@slorber/react-helmet-async@1.3.0`, and `react-loadable` resolves to `@docusaurus/react-loadable@6.0.0`. SPDX external references should capture the resolved aliased package names.

## Supply chain security notes

- **Private packages excluded from publication**: `argos`, `website`, and `test-bad-package` are marked `private: true`, reducing their attack surface on npm.
- **Public packages declare MIT**: Both published packages declare the MIT license and use `publishConfig.access: "public"`.
- **Node.js version floor**: Published packages enforce `node >=24.21`, limiting exposure to legacy runtime vulnerabilities.
- **Engine constraint propagation**: The website package has no `engines` field, but it depends on `@docusaurus/core`, which enforces the Node.js constraint transitively through peer and workspace dependency resolution.

## SBOM completeness limitations

The SPDX records in this document are derived from 5 pre-fetched `package.json` files. The following SBOM fields are not populated because the data is not available in the source context:

| SPDX field | Status | Reason |
|---|---|---|
| `PackageChecksum` | Absent | Requires resolved artifact hashes (SHA256/SHA1) |
| `PackageDownloadLocation` | `NOASSERTION` | Requires npm registry URLs |
| `PackageVerificationCode` | Absent | Requires file-level analysis |
| `FilesAnalyzed` | `false` | No source file enumeration provided |
| `relationship` for all transitive dependencies | Partial | Only 5 manifests available; lockfile not included |

For a complete SPDX SBOM, generate the document from the repository lockfile (`pnpm-lock.yaml`) with a tool such as `@cyclonedx/cyclonedx-npm` or Syft, which produces resolved versions, checksums, and the full transitive dependency graph.