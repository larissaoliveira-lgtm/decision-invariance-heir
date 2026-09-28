# Decision Invariance under Homomorphic Inference

Open-science repository for evaluating whether a real CKKS homomorphic-inference pipeline, compiled with HEIR and executed with OpenFHE, preserves model outputs and downstream operational decisions under a fixed, pre-specified three-action policy.

The study distinguishes numerical approximation from decision-level consequences. Rather than requiring exact floating-point equality between plaintext and homomorphic inference, it evaluates whether approximation introduced across the `float64 -> float32 -> HE` pipeline remains sufficiently small to preserve downstream actions under a frozen operational policy.

> **Current phase:** CONFIRMATORY execution completed for the main workload.
>
> The main locked-test run was executed from the frozen v1.3 PILOT state with confirmatory freeze SHA-256 `3b0347b70e8df2979b4371ee1fecd273d79f197c498d42afea526a0b2c11e45e`. The confirmatory run ID is `20260917T195635259936Z`.
>
> The operational policy used in this study is an experimental methodological construct for evaluating decision invariance. It is not intended to constitute a clinical decision rule.

## Research question

Under a fixed, pre-specified operational policy, to what extent does a real HEIR/OpenFHE CKKS inference pipeline preserve model outputs within pre-specified numerical tolerances and downstream operational decisions relative to plaintext inference, and what portion of any observed numerical discrepancy is attributable to float32 model export versus homomorphic evaluation?

## Pre-specified analytical objectives, hypotheses, and diagnostic criteria

The study evaluates four pre-specified analytical objectives addressing complementary aspects of numerical fidelity and decision preservation under homomorphic inference.

### H1 — Coexistence of binary-prediction preservation and operational-decision divergence

H1 evaluates whether binary model predictions remain fully preserved while at least one downstream operational action differs between plaintext and homomorphic inference.

The criterion is satisfied only when both conditions hold simultaneously:

1. the number of binary-classification disagreements is exactly zero; and
2. the number of operational-action disagreements is at least one.

H1 is treated as a **pre-specified composite empirical criterion**, not as a standalone null-hypothesis significance test.

### H2 — Concentration of operational disagreements near decision boundaries

H2 evaluates whether operational-action disagreements are disproportionately concentrated among observations close to the pre-specified operational decision boundaries.

Near-boundary status is derived from the plaintext operational margin using a cutoff determined exclusively from calibration data. The primary definition uses the pre-specified calibration quantile `q = 0.10`, with `q = 0.05` and `q = 0.20` retained as sensitivity analyses.

The two primary configuration-specific H2 tests are adjusted using the Holm procedure. H2 is not estimable when no operational disagreements occur or when the required near/far strata are empty.

### H3 — Mechanistic diagnostic value of error relative to operational margin

H3 evaluates whether numerical error interpreted relative to the available operational decision margin provides greater diagnostic discrimination of operational boundary crossings than raw numerical error alone.

The mechanistically informed diagnostic score is based on

\[
\frac{\text{numerical error}}{\text{operational margin}}.
\]

Because the relationship among numerical error, decision margin, and threshold crossing is partly structural, H3 is treated as a **mechanistic diagnostic analysis rather than as an independent confirmatory hypothesis-testing family**.

### H4 — Configuration-dependent incremental homomorphic error

H4 evaluates whether the two pre-specified CKKS configurations differ in the magnitude of numerical error introduced specifically by homomorphic evaluation.

The primary endpoint is the paired difference in incremental homomorphic absolute error relative to the float32-export plaintext computation. The comparison uses paired bootstrap inference and a paired sign-flip randomization procedure according to the frozen protocol.

### Multiplicity scope

- **H1** is a composite empirical criterion.
- **H2** contains two configuration-specific confirmatory tests and controls family-wise error across those tests using Holm adjustment.
- **H3** is diagnostic.
- **H4** is a distinct pre-specified confirmatory endpoint with a different estimand.

The study does not claim omnibus family-wise error control across the heterogeneous H1–H4 collection.

## Evidence labels

The repository keeps evidence classes separated according to their scientific and inferential roles:

- `SIMULATED_METHOD_VALIDATION` — methodology-development evidence only.
- `HEIR_REAL_HE_PILOT_CALIBRATION` — real-HE calibration, feasibility, and protocol-freezing evidence only.
- `HEIR_BOUNDARY_STRESS_DIAGNOSTIC` — mechanistic stress evidence only and **not** a population event-rate estimate.
- `HEIR_REAL_HE_CONFIRMATORY` — locked-test confirmatory evidence generated after the frozen execution gate passed.

## Study design

The study uses a fixed, provenance-preserving allocation of the scikit-learn Wisconsin Breast Cancer benchmark:

- training: 341 cases;
- calibration: 114 cases;
- locked test: 114 cases;
- split seeds: 42 / 43.

The locked test is a pre-specified held-out subset and is **not an external validation cohort**. Its allocation and identifiers are verified using frozen SHA-256 provenance records.

Locked-test features and labels are not used for model fitting, calibration, CKKS-configuration selection, boundary-definition tuning, operational-threshold tuning, plausibility-threshold tuning, or exploratory robustness analysis.

Two CKKS configurations were frozen before confirmatory access:

1. `ckks_reference_f55_s45`
2. `ckks_reduced_f50_s40`

No configuration was selected, modified, discarded, or re-parameterized based on locked-test outcomes.

## Numerical error decomposition

Let \(f_{64}(x)\) denote the float64 plaintext reference output, \(f_{32}(x)\) the float32-export plaintext output, and \(f_{\mathrm{HE}}(x)\) the homomorphic-inference output.

The float64-to-float32 model-export deviation is

\[
E_{\mathrm{export}}(x)=f_{32}(x)-f_{64}(x).
\]

The incremental deviation introduced by homomorphic evaluation is

\[
E_{\mathrm{HE}}(x)=f_{\mathrm{HE}}(x)-f_{32}(x).
\]

The total end-to-end deviation is

\[
E_{\mathrm{total}}(x)=f_{\mathrm{HE}}(x)-f_{64}(x),
\]

with

\[
E_{\mathrm{total}}(x)=E_{\mathrm{export}}(x)+E_{\mathrm{HE}}(x).
\]

This decomposition prevents numerical deviations introduced during model export from being incorrectly attributed to CKKS homomorphic evaluation.

## Decision invariance

CKKS implements approximate arithmetic. Therefore, the study does not require exact numerical equality between plaintext and homomorphic outputs.

**Decision invariance** refers to preservation of the downstream operational action under the pre-specified policy rather than to bitwise or exact floating-point equality.

The analysis distinguishes among numerical output deviation, operational decision margin, threshold crossing, binary-classification disagreement, and downstream operational-action disagreement.

## Main confirmatory results

The main confirmatory evaluation used the frozen locked test of 114 observations and evaluated both pre-specified CKKS configurations.

| Result | `ckks_reference_f55_s45` | `ckks_reduced_f50_s40` |
|---|---:|---:|
| Plaintext binary accuracy | 0.982456 | 0.982456 |
| HE binary accuracy | 0.982456 | 0.982456 |
| Binary-classification disagreements | 0 | 0 |
| Operational-action disagreements | 0 | 0 |
| Total output MAE vs float64 | 9.6331e-07 | 9.1530e-07 |
| Incremental HE MAE vs float32 export | 1.2192e-06 | 1.2573e-06 |

Within this locked test, both configurations preserved all binary classifications and all three-action operational decisions. Because no operational disagreement was observed, **H1's strict coexistence criterion was not met**, **H2 was not estimable**, and **H3 was not estimable**.

For H4, the paired difference in incremental HE MAE (`reduced - reference`) was `3.8021e-08`, with 95% paired-bootstrap CI `[-8.7319e-08, 1.6862e-07]`; the two-sided paired sign-flip randomization p-value was `0.5717`. The frozen analysis therefore did not detect a configuration-dependent difference for the primary H4 endpoint.

Zero observed disagreements do **not** establish universal invariance. With `n = 114`, the exact one-sided 95% upper bound for a zero-event disagreement rate is approximately `0.02594` (2.59%).

Machine-readable results are available under `results/confirmatory/`, and the complete confirmatory artifact is distributed as a release asset.

## Robustness and statistical analysis

The frozen protocol includes:

- decomposition of `float64 -> float32 -> HE` numerical error;
- exact finite-sample confidence intervals and one-sided upper bounds for rare disagreement events;
- paired bootstrap inference;
- paired sign-flip randomization for H4;
- Holm adjustment for the two primary configuration-specific H2 tests;
- pre-specified sensitivity analyses for near-boundary definitions;
- pre-specified sensitivity analyses for operational-policy thresholds;
- calibration-derived boundary-stress cases;
- plausibility diagnostics using marginal-support checks, Ledoit-Wolf shrinkage Mahalanobis distance, and k-nearest-neighbor distance;
- repeat-level numerical-stability diagnostics;
- cryptographic and runtime provenance;
- artifact hashing;
- transitive scientific-code fingerprinting;
- a frozen scientific-code bundle digest.

Boundary-stress cases are intentionally enriched for observations with small operational margins and are used for **mechanistic robustness analysis only**.

## Integrity and reproducibility safeguards

The study incorporates:

- immutable split provenance verified by SHA-256 identities;
- a transitive notebook-function manifest covering frozen scientific-analysis functions and referenced notebook-defined helper functions;
- a SHA-256 digest of the complete reproducibility bundle;
- frozen model, policy, thresholds, CKKS configurations, robustness specifications, statistical procedures, and analysis code;
- environment provenance covering Python, operating system, architecture, libc/loader evidence, dependencies, binaries, and runtime libraries;
- a strict compatibility contract for confirmatory execution;
- exact finite-sample treatment of rare events.

The main confirmatory artifact was audited against the frozen PILOT state. The freeze digest and frozen protocol files match the current preserved v1.3 PILOT records, all 74 artifacts referenced by the freeze record match their expected SHA-256 values, and all 249 entries in the confirmatory bundle manifest verify successfully. See `docs/confirmatory-audit.md` for the audit record and its limitations.

## Repository layout

```text
.
├── README.md
├── CITATION.cff
├── MANIFEST.pilot.sha256.json
├── MANIFEST.confirmatory.sha256.json
├── notebooks/
│   ├── OpenScience_HEIR_Decision_Invariance_PILOT_v1_3.ipynb
│   └── OpenScience_HEIR_Decision_Invariance_CONFIRMATORY_v1_3.ipynb
├── protocol/
├── environment/
│   └── confirmatory/
├── data/
├── models/
├── results/
│   ├── pilot/
│   └── confirmatory/
├── figures/
│   ├── pilot/
│   └── confirmatory/
├── replication/
│   └── banknote/
├── docs/
│   ├── methodology.md
│   ├── reproducibility.md
│   ├── artifact-integrity.md
│   ├── confirmatory-audit.md
│   └── open-science-checklist.md
├── scripts/
│   ├── validate_notebook.py
│   └── verify_sha256.py
└── .github/workflows/
    └── validate.yml
```

## Reproduction workflow

### 1. PILOT / calibration and freeze

Run `notebooks/OpenScience_HEIR_Decision_Invariance_PILOT_v1_3.ipynb` in a fresh compatible Linux/Colab runtime. The complete frozen PILOT reproducibility bundle is distributed as a release asset.

The PILOT workflow establishes the fixed split, model, operational policy, two CKKS configurations, calibration-derived robustness specifications, environment state, scientific-code fingerprints, and confirmatory-freeze digest.

### 2. CONFIRMATORY / locked test

Run `notebooks/OpenScience_HEIR_Decision_Invariance_CONFIRMATORY_v1_3.ipynb` in a fresh compatible runtime using the complete frozen PILOT bundle and the independently preserved freeze digest.

Before locked-test materialization, the workflow verifies the frozen scientific state, split provenance, policy, CKKS configurations, dependencies, runtime libraries, environment compatibility contract, artifact hashes, and confirmatory analysis code.

The execution gate fails closed if any required condition is not satisfied.

## Release assets

Large reproducibility bundles are intentionally kept outside normal Git history and should be attached to a tagged GitHub Release. Preserve each archive together with its SHA-256 sidecar.

## Publication metadata

The repository URL and public software/documentation license must be populated before final public release. No license is inferred automatically by the artifact pipeline.
