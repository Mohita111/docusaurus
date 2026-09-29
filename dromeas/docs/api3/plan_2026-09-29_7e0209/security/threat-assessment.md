# Threat Assessment: Docusaurus Documentation Platform

## Purpose

This threat assessment identifies, categorizes, and ranks security risks in the `Mohita111/docusaurus` repository. It focuses on the documentation site build pipeline, development tooling, and the dependency supply chain.

## Scope and methodology

This assessment uses the STRIDE model to analyze the repository. The evidence source is limited to the following package manifest files:

- `website/package.json`
- `packages/docusaurus/package.json`
- `packages/lqip-loader/package.json`
- `argos/package.json`
- `admin/test-bad-package/package.json`

Runtime source files such as plugin implementations, theme components, and webpack configuration were not available in this context. Risks that require inspection of implementation logic are marked as *unverified*.

## System description

The repository is a Docusaurus monorepo. It contains:

- `@docusaurus/core` — the static site generator core, including the CLI, webpack bundling, and dev server tooling
- `website` — the production documentation website with plugins for PWA, Google gtag, Mermaid, KaTeX math, Ideal Image, and live code blocks
- `@docusaurus/lqip-loader` — a webpack loader that generates low-quality image placeholders using `sharp`
- `argos` — Playwright-based visual regression testing uploaded to Argos CI
- `admin/test-bad-package` — a private fixture that intentionally pins React 16.14.0 and `@mdx-js/react` 1.0.1

## Assets requiring protection

| Asset | Location | Security properties needed |
| --- | --- | --- |
| Documentation source content | `website/` | Integrity, availability |
| Translations | Crowdin external service | Confidentiality of API token, integrity of translated content |
| Build output | `website/build` | Integrity, authenticity |
| End-user browser sessions | Website visitors | Privacy, integrity of served assets |
| CI/CD pipelines | Netlify, Argos, Crowdin integrations | Confidentiality of deployment and upload tokens |
| npm dependency supply chain | All package manifests | Integrity of resolved packages |

## Trust boundaries

```
Content author
    │
    ▼
Git repository ──► Netlify CI/CD ──► Build artifacts ──► Static hosting ──► End user
    │                                      │
    ├── Crowdin API ─── translation data ──┤
    └── npm registry ─── dependencies ─────┘
```

The main trust boundaries are:

1. Between local development environments and the npm package registry
2. Between the CI/CD runner and external services such as Crowdin, Argos, and Netlify
3. Between build-time code execution and the generated static output
4. Between the dev server and visitors on the local network

## Threat actors

| Actor | Motivation | Capabilities |
| --- | --- | --- |
| Malicious npm maintainer | Supply chain compromise | Publish malicious versions of any dependency |
| Compromised CI/CD account | Repository sabotage or data theft | Modify deployments, read tokens |
| Malicious content contributor | Content tampering | Submit translation changes or code changes |
| Local network attacker | Intercept dev server traffic | Observe or modify HTTP traffic to `webpack-dev-server` |
| Untrusted image submitter | Exploit native image processing | Supply crafted images to LQIP processing |

## Attack surface summary

| Entry point | Source manifest | Security-relevant dependency or script |
| --- | --- | --- |
| CLI build command | `packages/docusaurus/package.json` | `eval` `^0.1.8` |
| Local dev server | `packages/docusaurus/package.json` | `webpack-dev-server` `^6.0.0`, `serve-handler` `^6.1.7` |
| Bundle analysis | `packages/docusaurus/package.json` | `webpack-bundle-analyzer` `^5.3.2`, `open` `^11.0.2` |
| Native image processing | `packages/lqip-loader/package.json` | `sharp` `^0.35.1`, `file-loader` `^6.2.0` |
| Translation pipeline | `website/package.json` | `@crowdin/cli` `^5.0.2`, `@crowdin/crowdin-api-client` `^1.57.0` |
| Website analytics | `website/package.json` | `@docusaurus/plugin-google-gtag` `workspace:*` |
| Progressive web app | `website/package.json` | `@docusaurus/plugin-pwa`, `workbox-routing` `^7.4.1`, `workbox-strategies` `^7.4.1` |
| Visual regression upload | `argos/package.json` | `@argos-ci/cli` `^6.9.2`, `@argos-ci/playwright` `^7.5.0` |
| Browser automation | `argos/package.json` | `@playwright/test` `^1.63.0` |
| HTML parsing during tests | `argos/package.json` | `cheerio` `^1.2.0` |

## Threat catalog

### S1 — Malicious npm dependency substitution (spoofing)

Internal packages use `workspace:*` in `website/package.json`, but external dependencies rely on semver ranges such as `^0.1.8`, `^6.1.7`, and `^0.35.1`. A malicious version published within the allowed range could be installed without a manifest change.

**Evidence:** `packages/docusaurus/package.json` declares `eval: ^0.1.8`, `serve-handler: ^6.1.7`, and 25 other external version ranges. `website/package.json` declares 20 external version ranges.

**Impact:** High — arbitrary code execution during build or developer machine compromise.

**Likelihood:** Medium for low-maintenance packages; higher if lockfiles are not enforced.

### S2 — Shared dependency alias confusion (spoofing)

`@docusaurus/core` uses `react-helmet-async` and `react-loadable` npm aliases:

```json
"react-helmet-async": "npm:@slorber/react-helmet-async@1.3.0",
"react-loadable": "npm:@docusaurus/react-loadable@6.0.0"
```

Aliased packages resolve outside the expected package namespace. If the target package is abandoned or the namespace is compromised, the aliased dependency becomes a supply-chain risk.

**Evidence:** `packages/docusaurus/package.json`, dependencies block.

**Impact:** High — package compromise could inject malicious code into the React rendering pipeline.

**Likelihood:** Low — aliases point to forked copies under the project or maintainer namespace.

### T1 — Build-time code evaluation via `eval` package (tampering)

`@docusaurus/core` depends on the `eval` package `^0.1.8`. The package name suggests dynamic evaluation, but the exact call sites are not visible in the manifest.

**Evidence:** `packages/docusaurus/package.json`, `"eval": "^0.1.8"`.

**Impact:** High if it evaluates content-derived or external input — remote code execution during build.

**Likelihood:** Unverified — requires source inspection of `@docusaurus/core` consumers.

### T2 — Malicious Crowdin translations (tampering)

The website build downloads translations from Crowdin during Netlify builds. A compromised Crowdin project or API token could inject malicious translated content into the generated site.

**Evidence:** `website/package.json` scripts `netlify:crowdin:downloadTranslations` and `netlify:crowdin:downloadTranslationsFailSafe`.

**Impact:** High — stored XSS on all translated pages if translated content is not sanitized.

**Likelihood:** Medium — the translation service is an external trust boundary.

### T3 — Native image processing vulnerabilities in `sharp` (tampering / DoS)

`@docusaurus/lqip-loader` uses `sharp` `^0.35.1` to process images at build time. Malformed or malicious image files could trigger memory corruption or resource exhaustion.

**Evidence:** `packages/lqip-loader/package.json`, `"sharp": "^0.35.1"`.

**Impact:** Medium — build failure or, in rare cases, memory corruption during build.

**Likelihood:** Low to medium — depends on whether untrusted images are accepted.

### I1 — Secrets exposure through CI script output (information disclosure)

The `netlify:crowdin:downloadTranslationsFailSafe` script intentionally masks Crowdin failures but does not log the token. The comment in the script confirms that the Crowdin environment token is restricted to internal PRs:

```
(pnpm --dir .. crowdin:download:website || echo 'Crowdin translation download failure (only internal PRs have access to the Crowdin env token)')
```

**Evidence:** `website/package.json`, `netlify:crowdin:downloadTranslationsFailSafe`.

**Impact:** Medium if misconfigured — restricted token could leak through verbose logging in other paths.

**Likelihood:** Low — the current script seems designed to avoid token exposure.

### I2 — Bundle analyzer exposure on developer machines (information disclosure)

`webpack-bundle-analyzer` `^5.3.2` and `open` `^11.0.2` are present in `@docusaurus/core`. If the analyzer server binds to all interfaces, it can expose source maps or bundle structure to the local network.

**Evidence:** `packages/docusaurus/package.json`.

**Impact:** Low to medium — exposure of internal module paths and bundle contents.

**Likelihood:** Low — usually only run manually.

### D1 — Local dev server with insufficient network restrictions (denial of service / tampering)

`webpack-dev-server` `^6.0.0` and `serve-handler` `^6.1.7` serve the development site. A dev server bound to `0.0.0.0` without TLS allows local network attackers to observe or modify content in transit.

**Evidence:** `packages/docusaurus/package.json`.

**Impact:** Medium — content tampering or session observation during development.

**Likelihood:** Low for secure default configurations; unverified for the actual CLI bind address.

### D2 — Resource exhaustion from unbounded image processing (denial of service)

LQIP processing via `sharp` can be CPU- and memory-intensive. A repository that accepts large or malformed images without size limits can exhaust build runners.

**Evidence:** `packages/lqip-loader/package.json`.

**Impact:** Medium — stalled or killed CI builds.

**Likelihood:** Medium if image submission is automated or open to contributors.

### E1 — Privilege escalation through Playwright browser execution (elevation of privilege)

The `argos` package downloads and executes Playwright browser binaries. A compromised Playwright package or a malicious screenshot target can execute outside a sandboxed browser environment.

**Evidence:** `argos/package.json`, `@playwright/test` `^1.63.0`, `@argos-ci/playwright` `^7.5.0`.

**Impact:** Medium — arbitrary code execution on the test runner.

**Likelihood:** Low with Playwright's maintained browsers; higher if screenshots load untrusted URLs.

### E2 — Outdated React fixture used in MDX testing (elevation of privilege)

`admin/test-bad-package/package.json` pins `react` and `react-dom` to `16.14.0`. This fixture is private and appears to intentionally reproduce a bad dependency resolution scenario. If the fixture is imported by test infrastructure, the outdated React version may contain known vulnerabilities.

**Evidence:** `admin/test-bad-package/package.json`.

**Impact:** Low — limited to test environments.

**Likelihood:** Low — the package is `private: true` and named `test-bad-package`.

## Risk matrix

| Threat | Likelihood | Impact | Risk level |
| --- | --- | --- | --- |
| S1 — Malicious npm dependency substitution | Medium | High | **High** |
| S2 — Alias confusion | Low | High | **Medium** |
| T1 — Build-time `eval` package | Unverified | High | **Needs review** |
| T2 — Malicious Crowdin translations | Medium | High | **High** |
| T3 — Vulnerable `sharp` version | Low | Medium | **Low** |
| I1 — Secrets in CI output | Low | Medium | **Low** |
| I2 — Bundle analyzer exposure | Low | Medium | **Low** |
| D1 — Dev server network exposure | Low | Medium | **Low** |
| D2 — Image processing resource exhaustion | Medium | Medium | **Medium** |
| E1 — Playwright browser escape | Low | Medium | **Low** |
| E2 — Outdated React fixture | Low | Low | **Low** |

## Security controls observed

| Control | Location | Notes |
| --- | --- | --- |
| Workspace-internal dependency pinning | `website/package.json` | All `@docusaurus/*` packages use `workspace:*` |
| Private package flags | `website/package.json`, `argos/package.json` | Prevents accidental publishing |
| Conditional Crowdin token access | `website/package.json` | Failsafe script handles missing token without crashing |
| Node engine constraint | `packages/docusaurus/package.json`, `packages/lqip-loader/package.json` | Requires Node `>=24.21` |
| Dedicated bad-package fixture | `admin/test-bad-package/package.json` | Isolates dependency-resolution failure testing |

## Recommendations

1. **Pin external dependencies with exact versions or lockfiles.** Replace caret ranges such as `"sharp": "^0.35.1"` and `"eval": "^0.1.8"` with exact versions or enforce `pnpm-lock.yaml` in CI.

2. **Audit every call site of the `eval` package in `@docusaurus/core`.** Determine whether it processes untrusted, content-derived, or network-derived strings. Replace with a safer alternative if dynamic evaluation cannot be removed.

3. **Rotate and scope Crowdin, Netlify, and Argos tokens.** Use short-lived, read-only upload tokens where possible and restrict them to specific CI pipeline branches or preview builds.

4. **Configure `webpack-dev-server` and `serve-handler` to bind to localhost by default.** Avoid accepting connections from `0.0.0.0` in development mode.

5. **Apply image size and format allowlists in LQIP processing.** Reject oversized images, exotic formats, and files with missing MIME metadata before passing them to `sharp`.

6. **Scan the dependency tree with a software composition analysis tool.** Generate an SBOM in SPDX or CycloneDX format for each published package and run it during CI. Include `sharp`, `playwright`, `webpack-dev-server`, and `serve-handler` as high-priority components.

7. **Sign generated build artifacts and verify checksums in Netlify.** A signed artifact or content digest reduces the impact of a compromised intermediate step in the build pipeline.

## Limitations of this assessment

This assessment is derived only from package manifest files. It does not inspect runtime implementation details such as:

- Actual `eval` usage in `@docusaurus/core`
- Dev server bind address and default TLS configuration
- Content sanitization of Crowdin translations
- Image validation logic in `@docusaurus/lqip-loader`
- PWA caching behavior and service worker update policies

A complete threat model requires access to the source files for those packages, the webpack configurations, and the CI/CD environment variable definitions.