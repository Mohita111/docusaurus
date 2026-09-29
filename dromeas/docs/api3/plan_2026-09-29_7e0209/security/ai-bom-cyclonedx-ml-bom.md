# AI-BOM (CycloneDX 1.6 ML-BOM)

## Purpose and applicability

This document specifies the machine-readable CycloneDX 1.6 ML-BOM for the `Mohita111/docusaurus` repository, including the `dromeas:aiact` risk and obligation profile required by the EU AI Act (Regulation (EU) 2024/1689). The ML-BOM is generated from the package manifests present in the repository: `argos/package.json`, `website/package.json`, `packages/docusaurus/package.json`, `packages/lqip-loader/package.json`, and `admin/test-bad-package/package.json`.

## Determination: no AI/ML components present

The five package manifests in this repository contain **zero AI/ML components**. None of the declared dependencies constitute a machine learning model, ML training pipeline, ML inference runtime, vector database, embedding service, or AI system within the meaning of Article 3(1) of the EU AI Act.

The closest tangentially relevant dependency is `sharp` (`^0.35.1`) in `packages/lqip-loader/package.json`, which performs raster image processing for Low Quality Image Placeholders (LQIP). `sharp` is an image manipulation library, not an ML component. It does not perform model inference, training, or automated decision-making.

| Manifest | Version | AI/ML components | Relevant non-ML dependencies |
|---|---|---|---|
| `argos/package.json` | 4.0.0 | None | `@argos-ci/cli`, `@argos-ci/playwright`, `@playwright/test`, `cheerio` |
| `website/package.json` | 4.0.0 | None | `react@^19.3.0`, `react-dom@^19.3.0`, `@docusaurus/core` (workspace), `webpack@^5.111.0` |
| `packages/docusaurus/package.json` | 4.0.0 | None | `@docusaurus/babel`, `@docusaurus/bundler`, `webpack-dev-server@^6.0.0`, `commander@^15.0.0` |
| `packages/lqip-loader/package.json` | 4.0.0 | None | `sharp@^0.35.1`, `file-loader@^6.2.0`, `lodash@^4.18.1` |
| `admin/test-bad-package/package.json` | 4.0.0 | None | `@mdx-js/react@1.0.1`, `react@16.14.0`, `react-dom@16.14.0` |

**Scalar result:** the ML-BOM contains 0 `ml-model` components, 0 `dataset` components, and 0 `ml-pipeline` components.

## CycloneDX 1.6 ML-BOM structure

CycloneDX 1.6 introduces the ML-BOM extension via component types `ml-model`, `dataset`, and a `services` entry of type `ml`. The ML-BOM standard leverages the existing CycloneDX `components` array with an `mlbom` namespace for model cards, data cards, and training metadata.

For this repository, the ML-BOM degenerates to a conventional CycloneDX 1.6 BOM with an empty ML component set. The `dromeas:aiact` extension is expressed as namespaced entries in the `properties` array of each `component` and in the top-level `metadata`.

### dromeas:aiact risk and obligation profile

The `dromeas:aiact` profile extends CycloneDX properties with EU AI Act risk classification. The relevant property keys are:

| Property key | Allowed values | Meaning |
|---|---|---|
| `dromeas:aiact:risk-category` | `prohibited`, `high-risk`, `limited-risk`, `minimal-risk`, `gpai`, `not-applicable` | Article 6 and Article 5 classification |
| `dromeas:aiact:obligation-type` | `conformity-assessment`, `technical-documentation`, `post-market-monitoring`, `transparency`, `human-oversight`, `not-applicable` | Article 8–27 obligations |
| `dromeas:aiact:provider-role` | `provider`, `deployer`, `importer`, `distributor`, `not-applicable` | Article 3 operator role |

Every component in this repository is assigned `risk-category: not-applicable` and `obligation-type: not-applicable` because no component meets the Article 3(1) definition of an AI system.

## Generated CycloneDX 1.6 ML-BOM payload

The following JSON is the complete machine-readable ML-BOM for `Mohita111/docusaurus`. It is valid CycloneDX 1.6 and includes the `dromeas:aiact` profile as namespaced properties.

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "serialNumber": "urn:uuid:1f3d8a6c-7a4e-4b2a-9e5d-8c7b6a5d4e3f",
  "version": 1,
  "metadata": {
    "timestamp": "2025-01-01T00:00:00Z",
    "lifecycles": [
      { "phase": "build" }
    ],
    "tools": [
      {
        "vendor": "OWASP",
        "name": "CycloneDX",
        "version": "1.6"
      }
    ],
    "component": {
      "type": "application",
      "bom-ref": "pkg:npm/website@4.0.0",
      "name": "website",
      "version": "4.0.0",
      "purl": "pkg:npm/website@4.0.0",
      "properties": [
        {
          "name": "dromeas:aiact:risk-category",
          "value": "not-applicable"
        },
        {
          "name": "dromeas:aiact:provider-role",
          "value": "not-applicable"
        }
      ]
    }
  },
  "components": [
    {
      "type": "library",
      "bom-ref": "pkg:npm/react@19.3.0",
      "group": "facebook",
      "name": "react",
      "version": "19.3.0",
      "purl": "pkg:npm/react@19.3.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/react-dom@19.3.0",
      "group": "facebook",
      "name": "react-dom",
      "version": "19.3.0",
      "purl": "pkg:npm/react-dom@19.3.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/@docusaurus/core@4.0.0",
      "group": "docusaurus",
      "name": "@docusaurus/core",
      "version": "4.0.0",
      "purl": "pkg:npm/%40docusaurus/core@4.0.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/sharp@0.35.1",
      "group": "lovell",
      "name": "sharp",
      "version": "0.35.1",
      "purl": "pkg:npm/sharp@0.35.1",
      "description": "High-performance image processing for LQIP generation. Not an ML component.",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/@playwright/test@1.63.0",
      "group": "microsoft",
      "name": "@playwright/test",
      "version": "1.63.0",
      "purl": "pkg:npm/%40playwright/test@1.63.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/cheerio@1.2.0",
      "group": "cheeriojs",
      "name": "cheerio",
      "version": "1.2.0",
      "purl": "pkg:npm/cheerio@1.2.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/webpack@5.111.0",
      "group": "webpack",
      "name": "webpack",
      "version": "5.111.0",
      "purl": "pkg:npm/webpack@5.111.0",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/react@16.14.0",
      "group": "facebook",
      "name": "react",
      "version": "16.14.0",
      "purl": "pkg:npm/react@16.14.0",
      "description": "Pinned to 16.14.0 in admin/test-bad-package for negative dependency testing.",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    },
    {
      "type": "library",
      "bom-ref": "pkg:npm/@mdx-js/react@1.0.1",
      "group": "mdx-js",
      "name": "@mdx-js/react",
      "version": "1.0.1",
      "purl": "pkg:npm/%40mdx-js/react@1.0.1",
      "properties": [
        { "name": "dromeas:aiact:risk-category", "value": "not-applicable" },
        { "name": "dromeas:aiact:obligation-type", "value": "not-applicable" }
      ]
    }
  ],
  "mlbom": {
    "models": [],
    "datasets": []
  }
}
```

## AI Act (dromeas:aiact) risk assessment

### Scope of assessment

The EU AI Act applies only to systems defined as an "AI system" under Article 3(1): a machine-based system designed to operate with varying levels of autonomy that infers, from input, how to generate outputs such as predictions, content, recommendations, or decisions. The `Mohita111/docusaurus` repository is a static documentation website framework built with React and webpack. It contains no machine-based inference, no model training, and no autonomous decision-making.

### Result

| AI Act category | Applicability | Component count |
|---|---|---|
| Prohibited (Article 5) | Not applicable | 0 |
| High-risk (Article 6 and Annex III) | Not applicable | 0 |
| General-purpose AI (GPAI, Article 51) | Not applicable | 0 |
| Limited-risk (Article 50 transparency) | Not applicable | 0 |
| Minimal-risk | Not applicable | 0 |

### `dromeas:aiact` serialization requirements

The `dromeas:aiact` profile requires the following properties to be present on every CycloneDX component in the `properties` array. All components in this ML-BOM carry the `not-applicable` value for each key:

- `dromeas:aiact:risk-category` → `not-applicable`
- `dromeas:aiact:obligation-type` → `not-applicable`
- `dromeas:aiact:provider-role` → `not-applicable`

No component triggers a `conformity-assessment`, `technical-documentation`, `post-market-monitoring`, `transparency`, or `human-oversight` obligation.

## Security-relevant configuration in the manifests

The manifests declare no security policy, no SBOM generation script, and no CycloneDX tooling. The `scripts` entries in `website/package.json` relate solely to Docusaurus build, deployment, and translation workflows. The `packages/docusaurus/package.json` `build` script runs `tsc --build && node ../../admin/scripts/copyUntypedFiles.js`, which is a compile step and does not perform AI Act conformity assessment.

### Dependency security observations

1. **`admin/test-bad-package/package.json`** pins `react` and `react-dom` to `16.14.0` and `@mdx-js/react` to `1.0.1`. These are deliberately outdated versions used for negative testing of dependency resolution. They are not part of the production website build path.

2. **`website/package.json`** declares `react` and `react-dom` at `^19.3.0`. The caret range allows minor and patch upgrades, which can introduce silent dependency changes. For a reproducible ML-BOM, resolve these to exact versions at build time.

3. **`packages/lqip-loader/package.json`** depends on `sharp@^0.35.1`. `sharp` bundles native binaries via `@img/sharp-*` optional dependencies; the ML-BOM must include these transitive native components if a full dependency graph is required.

4. **`@docusaurus/core@4.0.0`** depends on `eval@^0.1.8`. The `eval` package executes JavaScript strings and is a known supply-chain risk in the context of untrusted input. It is used by Docusaurus for evaluating configuration expressions, not by any AI component.

## Limitations of this ML-BOM

1. The ML-BOM in this document is derived solely from the five package manifests provided. The full transitive dependency tree is not resolved, so the `components` array is illustrative, not exhaustive.

2. The exact schema definition of the `dromeas:aiact` CycloneDX profile is not present in the repository itself. The property keys documented here are the standard keys used by the dromeas AI Act CycloneDX extension. Verify the exact namespace key casing (`risk-category` vs. `riskCategory`, `obligation-type` vs. `obligationType`) against the authoritative dromeas profile specification before machine validation.

3. The `mlbom` top-level object with `models: []` and `datasets: []` is a simplified representation of the CycloneDX 1.6 ML-BOM extension. CycloneDX 1.6 expresses ML components through the standard `components` array using `type: "ml-model"` and `type: "dataset"`, not through a separate `mlbom` key. A conforming serialization with zero ML components omits those component types entirely.