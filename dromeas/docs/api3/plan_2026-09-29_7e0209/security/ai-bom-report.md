# AI-BOM Report

**Scope:** Mohita111/docusaurus repository  
**Documentation type:** Security  
**Sub-scope:** ai-bom  
**Status:** Agent-drafted — human review required  
**Date:** 2026-06-30T18:12:59.184Z

## 1. Executive summary

This AI Bill of Materials (AI-BOM) documents the component inventory, data flows, and EU AI Act applicability for the Docusaurus repository. Based on analysis of the five prefetched `package.json` files, this codebase is a static site generator and documentation tooling platform. **No AI/ML model components, training pipelines, inference engines, or generative AI systems were identified** in the analyzed source files.

The codebase consists of build tooling, web frameworks, testing utilities, and content plugins. The most security-relevant findings are a dynamic code evaluation dependency (`eval`) in `@docusaurus/core`, third-party analytics (Google gtag), and third-party translation processing (Crowdin).

Under the EU AI Act, this repository is assessed as **minimal risk** and does not trigger high-risk obligations under Annex III. However, if the generated documentation site publishes AI-generated content or integrates AI-powered search, Article 50 transparency obligations may apply.

## 2. Component inventory

The following components were extracted from the source `package.json` files.

### 2.1 Top-level workspaces

| Component | Version | Package file | Purpose |
|---|---|---|---|
| `argos` | 4.0.0 | `argos/package.json` | Visual regression testing using Argos CI |
| `website` | 4.0.0 | `website/package.json` | Docusaurus documentation website |
| `@docusaurus/core` | 4.0.0 | `packages/docusaurus/package.json` | Core Docusaurus static site generator |
| `@docusaurus/lqip-loader` | 4.0.0 | `packages/lqip-loader/package.json` | Low Quality Image Placeholder webpack loader |
| `test-bad-package` | 4.0.0 | `admin/test-bad-package/package.json` | Test fixture for dependency validation |

### 2.2 AI/ML-relevant assessment

**No AI or ML packages were found.** The following dependency categories were checked across all source files:

- **No** transformers, LLM, or NLP libraries
- **No** machine learning frameworks (TensorFlow, PyTorch, scikit-learn)
- **No** vector databases or embedding engines
- **No** AI inference runtimes or model servers
- **No** generative AI SDKs (OpenAI, Anthropic, LangChain)
- **No** AI model weights or training data

The codebase is a **non-AI system** in terms of embedded components.

### 2.3 Security-relevant dependencies

| Dependency | Version | Source | Security consideration |
|---|---|---|---|
| `eval` | ^0.1.8 | `packages/docusaurus/package.json` | Dynamic JavaScript code evaluation. Requires review for code injection surfaces. |
| `sharp` | ^0.35.1 | `packages/lqip-loader/package.json` | Native image processing module. Binary supply chain risk. |
| `cheerio` | ^1.2.0 | `argos/package.json` | HTML parsing used in test tooling. |
| `@docusaurus/plugin-google-gtag` | workspace:* | `website/package.json` | Third-party analytics. Sends visitor data to Google. |
| `@crowdin/cli` | ^5.0.2 | `website/package.json` | Translation management. Uploads source content to Crowdin cloud. |
| `@crowdin/crowdin-api-client` | ^1.57.0 | `website/package.json` | Crowdin REST API client. |
| `@playwright/test` | ^1.63.0 | `argos/package.json` | Browser automation. Handle test fixtures with care. |
| `@argos-ci/cli` | ^6.9.2 | `argos/package.json` | Uploads screenshots to Argos cloud service. |
| `@argos-ci/playwright` | ^7.5.0 | `argos/package.json` | Playwright integration for Argos visual testing. |
| `search-insights` | ^2.17.3 | `website/package.json` (devDep) | Algolia search analytics integration. |

## 3. Data flows

Data flows are inferred from dependency usage and script definitions in the source files. Those marked **external** transmit data off-machine.

### 3.1 Build-time data flows

| Flow | Path | Data type | Security relevance |
|---|---|---|---|
| Source content → webpack build | `website/**` → `docusaurus build` → static HTML/CSS/JS | Markdown, MDX, images | No PII. Output is public by design. |
| Translation upload | Build source → Crowdin API | Documentation source text | External. Crowdin receives content. Token required. |
| Translation download | Crowdin API → build output | Translated strings | External. Crowdin data ingested. |
| Visual testing upload | Playwright screenshots → Argos cloud | Screenshots of rendered pages | External. Screenshots may contain test data. |
| LQIP generation | Source images → `sharp` → placeholder output | Image data | Local processing. No external transmission. |

### 3.2 Runtime data flows (deployed website)

| Flow | Path | Data type | Security relevance |
|---|---|---|---|
| Visitor → site hosting | Browser → Netlify/CDN | HTTP requests, IP addresses | Standard hosting. |
| Analytics tracking | Browser → Google Analytics (via gtag plugin) | Page views, interaction events, browser metadata | External. Google receives visitor data. Configure with privacy policy. |
| Search | Browser → Algolia (via search-insights) | Search queries, click events | External. Algolia receives query data. |
| Service worker caching | Browser cache → site assets | Cached HTML/JS/CSS | Local. `workbox-routing` and `workbox-strategies` manage offline cache. |

### 3.3 CLI tool data flows

| Flow | Path | Data type | Security relevance |
|---|---|---|---|
| Update check | `update-notifier` → npm registry | Package version info | External. May send package metadata. |
| Bundle analysis | `webpack-bundle-analyzer` → local UI | Build statistics | Local only. |
| Port detection | `detect-port` | Local network | Local only. |

### 3.4 Dynamic code evaluation flow

`@docusaurus/core` includes the `eval` package as a direct runtime dependency. This represents a **code evaluation data flow**: JavaScript strings passed to `eval()` execute in the Node.js process context. Human review must confirm that no untrusted input reaches this evaluation surface.

## 4. EU AI Act obligations assessment

### 4.1 Risk classification

Based on the component inventory and data flows:

| EU AI Act category | Applicability | Rationale |
|---|---|---|
| Article 5 — Prohibited AI practices | Not applicable | No social scoring, biometric manipulation, or subliminal techniques identified. |
| Article 6 — High-risk AI systems (Annex III) | Not applicable | Static site generation and documentation tooling do not fall under critical infrastructure, education, employment, essential services, law enforcement, migration, or justice categories. |
| Article 50 — Transparency obligations | **Potentially applicable** | Applies only if AI-generated content is published on the generated site or AI-powered chatbots/interfaces are deployed. Not inherent to the codebase. |
| Article 52/53 — General-purpose AI (GPAI) | Not applicable | No general-purpose AI models included. |
| Article 95 — Codes of conduct | Voluntary | May adopt voluntarily for quality assurance. |

### 4.2 Summary of obligations

**Mandatory:** None identified from the analyzed source files.

**Conditional:** If the Docusaurus-generated documentation site incorporates AI features (for example, AI-generated documentation content, AI-powered search summaries, or chatbot widgets), then:

1. **Article 50(1):** Machine-generated content must be marked in a machine-readable format and disclosed as artificially generated or manipulated.
2. **Article 50(2):** Users interacting with an AI system must be informed unless obvious from context.
3. **Article 50(4):** Deepfake disclosures apply if synthetic media is published.

**Recommended voluntary measures:**

1. Maintain an up-to-date SBOM in CycloneDX or SPDX format, regenerated on each dependency change.
2. Document AI-generated content workflows if AI-assisted authoring is used for documentation.
3. Include a transparency notice in the site footer if AI search or summarization is enabled.

## 5. SBOM format requirements

This AI-BOM is derived from package manifest files and does not currently have a machine-readable SBOM counterpart. For compliance and supply chain security, generate SBOM artifacts as follows:

| Requirement | Recommended format | Practical implementation |
|---|---|---|
| Dependency inventory | CycloneDX 1.5 or SPDX 3.0 | Run a lockfile-based SBOM generator in CI. |
| Transitive dependencies | Lockfile (e.g., `pnpm-lock.yaml`) | Include resolved dependency graph, not only direct dependencies. |
| AI model inventory | CycloneDX ML-BOM profile | Not currently needed — no AI models present. |
| Component provenance | PURL (package URL) | Use `pkg:npm/<name>@<version>` identifiers. |
| Vulnerability tracking | VEX (Vulnerability Exploitability eXchange) | Supplement SBOM with VEX statements. |

**Current limitation:** Only five direct `package.json` files were available for analysis. Full transitive dependency resolution requires the lockfile and the remaining workspace `package.json` files.

## 6. Open items requiring human review

The following items are flagged for human verification. This report is agent-drafted and must not be treated as a final compliance document.

1. **`eval` dependency usage.** Confirm where and how the `eval` package is invoked in `@docusaurus/core`. Verify that no user-supplied or network-sourced strings reach `eval()`. If present, document the trust boundary.

2. **Google Analytics configuration.** Confirm whether `@docusaurus/plugin-google-gtag` is enabled in the Docusaurus configuration file. If enabled, verify that a privacy policy and consent mechanism are in place for EU/EEA visitors under GDPR.

3. **Crowdin token handling.** Confirm that Crowdin API tokens are managed as secrets and not committed to the repository. Review the `netlify:crowdin:uploadSources` and `netlify:crowdin:downloadTranslations` script paths for credential exposure.

4. **Argos screenshot data.** Confirm that Argos visual test screenshots do not include secrets, production data, or user PII. Review the `upload` script that sends `./screenshots/chromium` to the Argos cloud service.

5. **Test fixture package.** `admin/test-bad-package/package.json` pins `react@16.14.0` and `react-dom@16.14.0`. These versions may contain known vulnerabilities. Confirm this package is excluded from production builds and vulnerability scans.

6. **Full repository scope.** Only five source files were available. A complete AI-BOM requires all workspace `package.json` files, the root `package.json`, and lockfiles.

7. **AI-generated content workflow.** Confirm whether AI-assisted authoring tools are used to produce documentation content. If yes, establish an Article 50 transparency procedure.

8. **Node.js engine requirement.** All workspace packages specify `"node": ">=24.21"`. Confirm that deployment environments satisfy this requirement and that no older Node.js runtimes are used in production.

## 7. Methodology and limitations

**Methodology:** The component inventory was extracted directly from the five prefetched `package.json` files listed in Section 2.1. Data flows were inferred from dependency purposes, script names, and documented package behavior.

**Limitations:**

- Only direct dependencies are cataloged. Transitive dependencies require lockfile analysis.
- No runtime source code was analyzed beyond manifest declarations.
- The `@docusaurus/core` dependencies include the `eval` package, but the specific call sites were not available in the provided source files.
- EU AI Act assessments are preliminary and do not constitute legal advice.

**Recommended next steps:**

1. Generate a full SBOM using a lockfile-aware tool (for example, `pnpm audit` or CycloneDX generator).
2. Run a static code scan to locate all `eval()` call sites.
3. Review the Docusaurus configuration for plugin activation status.
4. Conduct a GDPR data flow assessment for Google Analytics and Crowdin integrations.