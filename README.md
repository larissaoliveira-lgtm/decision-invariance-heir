# Decision Invariance under Homomorphic Inference

Open-science repository for evaluating whether a real CKKS homomorphic-inference pipeline compiled with HEIR and executed with OpenFHE preserves model outputs and downstream operational decisions under a fixed three-action policy.

> **Current phase:** PILOT/calibration and protocol freeze. Confirmatory claims may only be produced after the complete frozen PILOT bundle has been restored in a fresh runtime, its integrity has been independently verified, and the confirmatory execution gate has passed.

## Research question

Under a fixed, pre-specified operational policy, to what extent does a real HEIR/OpenFHE CKKS inference pipeline preserve model outputs within pre-specified numerical tolerances and downstream operational decisions relative to plaintext inference, and what portion of any observed numerical discrepancy is attributable to float32 model export versus homomorphic evaluation?

## Pre-specified hypotheses and analytical criteria

The study evaluates four pre-specified hypotheses or analytical criteria addressing complementary aspects of decision preservation under homomorphic inference.

### H1 — Coexistence of binary-prediction preservation and operational-decision divergence

H1 evaluates whether binary model predictions remain fully preserved while at least one downstream operational action differs between plaintext and homomorphic inference.

The criterion is satisfied only when both of the following conditions hold simultaneously:

1. the number of binary classification disagreements is exactly zero; and
2. the number of operational-action disagreements is at least one.

H1 is evaluated as a **pre-specified composite empirical criterion**, rather than through a standalone null-hypothesis significance test. Aggregate binary-accuracy difference and classification-disagreement rate are reported separately as descriptive measures of predictive fidelity. No application-independent non-inferiority margin is introduced after locked-test access.

### H2 — Concentration of operational disagreements near decision boundaries

H2 evaluates whether operational-action disagreements are disproportionately concentrated among observations close to the pre-specified operational decision boundaries.

Near-boundary status is derived from the plaintext operational margin using a cutoff determined exclusively from calibration data. The primary definition uses the pre-specified calibration quantile \(q=0.10\), with \(q=0.05\) and \(q=0.20\) retained as sensitivity analyses.

The two primary configuration-specific H2 tests are adjusted using the Holm procedure. H2 is not estimable when no operational disagreements occur or when the required near/far strata are empty.

### H3 — Mechanistic diagnostic value of error relative to operational margin

H3 evaluates whether scaling numerical error by the available operational decision margin provides greater diagnostic discrimination of operational boundary crossings than raw numerical error alone.

The mechanistically informed diagnostic score is based on:

$$
\frac{\text{numerical error}}{\text{operational margin}}.
$$

Its ability to rank observations according to operational disagreement is compared with that of raw numerical error using diagnostic discrimination measures and bootstrap uncertainty.

Because the relationship between numerical error, decision margin, and threshold crossing is partly structural, H3 is treated as a **diagnostic analysis rather than an independent confirmatory hypothesis-testing family**. No multiplicity-adjusted reject/accept claim is made for H3.

### H4 — Configuration-dependent incremental homomorphic error

H4 evaluates whether the two pre-specified CKKS configurations differ in the magnitude of numerical error introduced specifically by homomorphic evaluation.

The primary endpoint is the paired difference in incremental homomorphic absolute error relative to the float32-export plaintext computation. This separates the HE-specific contribution from the common float64-to-float32 model-export component.

The comparison is evaluated using paired bootstrap inference and a paired sign-flip randomization procedure. Total float64-to-homomorphic error is treated as a secondary endpoint because it also contains the common model-export contribution.

### Multiplicity scope

H2 and H4 are distinct pre-specified confirmatory endpoints with different estimands. H2 controls family-wise error across its two primary configuration-specific tests using Holm adjustment. H3 is diagnostic, and H1 is a composite empirical criterion rather than a standalone hypothesis test.

Accordingly, the study does not claim omnibus family-wise error control across the heterogeneous H1–H4 collection.

## Evidence labels

The repository keeps evidence classes explicitly separated according to their inferential role:

* `SIMULATED_METHOD_VALIDATION` — methodology-development evidence only.
* `HEIR_REAL_HE_PILOT_CALIBRATION` — real-HE calibration and feasibility evidence only.
* `HEIR_BOUNDARY_STRESS_DIAGNOSTIC` — mechanistic stress evidence only and **not** a population event-rate estimate.
* `HEIR_REAL_HE_CONFIRMATORY` — the only evidence class supporting locked-test confirmatory claims.

## Study design

The study uses a fixed, provenance-preserving allocation of the scikit-learn Wisconsin Breast Cancer benchmark:

* training: 341 cases;
* calibration: 114 cases;
* locked test: 114 cases;
* split seeds: 42 / 43.

The locked test is a pre-specified provenance-preserving held-out subset and is **not an external validation cohort**. Its allocation and identifiers are verified using pre-recorded SHA-256 digests before confirmatory analysis.

Locked-test features and labels are not used for model fitting, calibration, CKKS-configuration selection, boundary-definition tuning, operational-threshold tuning, plausibility-threshold tuning, or exploratory robustness analysis.

Two CKKS configurations are pre-specified and frozen before confirmatory access:

1. `ckks_reference_f55_s45`
2. `ckks_reduced_f50_s40`

No configuration may be selected, modified, or discarded based on locked-test outcomes.

## Numerical error decomposition

The numerical evaluation reports two incremental sources of deviation and their total end-to-end effect.

Let \(f_{64}(x)\) denote the float64 plaintext reference output, \(f_{32}(x)\) the float32-export plaintext output, and \(f_{\mathrm{HE}}(x)\) the homomorphic-inference output.

The float64-to-float32 model-export deviation is:

$$
E_{\mathrm{export}}(x)
=
f_{32}(x)-f_{64}(x)
$$

The incremental deviation introduced by homomorphic evaluation is:

$$
E_{\mathrm{HE}}(x)
=
f_{\mathrm{HE}}(x)-f_{32}(x)
$$

The total end-to-end deviation is:

$$
E_{\mathrm{total}}(x)
=
f_{\mathrm{HE}}(x)-f_{64}(x)
$$

By construction:

$$
E_{\mathrm{total}}(x)
=
E_{\mathrm{export}}(x)
+
E_{\mathrm{HE}}(x)
$$

because

$$
\begin{aligned}
E_{\mathrm{export}}(x)+E_{\mathrm{HE}}(x)
&=
\left[f_{32}(x)-f_{64}(x)\right]
+
\left[f_{\mathrm{HE}}(x)-f_{32}(x)\right] \\
&=
f_{\mathrm{HE}}(x)-f_{64}(x).
\end{aligned}
$$

This decomposition prevents numerical deviations introduced during model export from being incorrectly attributed to CKKS homomorphic evaluation.

## Decision invariance

Because CKKS implements approximate arithmetic, the study does not assume exact numerical equality between plaintext and homomorphic outputs.

The primary operational question is whether numerical perturbations introduced by the homomorphic-inference pipeline are sufficiently small to preserve downstream decisions under the frozen operational policy.

The analysis therefore distinguishes among:

* numerical output deviation;
* operational decision margin;
* threshold crossing;
* binary-classification disagreement;
* downstream operational-action disagreement.

Decision invariance refers to preservation of the operational decision under the pre-specified policy rather than bitwise or exact floating-point equality.

## Robustness and statistical analysis

The protocol includes:

* decomposition of `float64 -> float32 -> HE` numerical error;
* exact finite-sample confidence intervals and one-sided upper bounds for rare disagreement events;
* paired bootstrap inference;
* paired sign-flip randomization for H4;
* Holm adjustment for the two primary configuration-specific H2 tests;
* pre-specified sensitivity analyses for near-boundary definitions;
* pre-specified sensitivity analyses for operational-policy thresholds;
* calibration-derived boundary stress cases;
* plausibility diagnostics for boundary-stress points using marginal-support checks, Ledoit-Wolf shrinkage Mahalanobis distance, and k-nearest-neighbor distance;
* repeat-level numerical-stability diagnostics;
* cryptographic and runtime provenance;
* artifact hashing;
* transitive scientific-code fingerprinting;
* a frozen scientific-code bundle digest.

Boundary-stress cases are intentionally enriched for observations with small operational margins. They are therefore used for **mechanistic robustness analysis only** and must not be interpreted as estimates of population disagreement frequency.

## Integrity and reproducibility safeguards

The study incorporates explicit safeguards designed to preserve the pre-specified scientific protocol across calibration and confirmatory execution.

These include:

* immutable split provenance verified by SHA-256 identities;
* a transitive notebook-function manifest covering frozen scientific analysis functions and referenced notebook-defined helper functions;
* a SHA-256 digest of the complete reproducibility bundle;
* environment provenance recording Python implementation and full version, operating system, platform, architecture, libc/loader evidence, dependencies, binaries, and relevant runtime libraries;
* a strict compatibility contract for confirmatory execution;
* frozen model, policy, thresholds, CKKS configurations, robustness specifications, statistical procedures, and analysis code;
* exact finite-sample treatment of rare events;
* independently preserved confirmation of the freeze digest.

These safeguards are established before locked-test access.

## Repository layout

```text
.
├── notebooks/
│   ├── OpenScience_HEIR_Decision_Invariance_PILOT.ipynb
│   └── OpenScience_HEIR_Decision_Invariance_CONFIRMATORY.ipynb
├── protocol/       # frozen protocol and confirmatory freeze record
├── environment/    # resolved environment and compatibility records
├── results/        # generated PILOT and CONFIRMATORY outputs
├── figures/        # generated plots
├── docs/
│   ├── methodology.md
│   ├── reproducibility.md
│   ├── artifact-integrity.md
│   └── open-science-checklist.md
├── scripts/
│   ├── validate_notebook.py
│   └── verify_sha256.py
├── .github/workflows/
│   └── validate.yml
├── CITATION.cff
├── CONTRIBUTING.md
├── SECURITY.md
└── .gitignore
```

## Reproduction workflow

### 1. PILOT / calibration

Run the PILOT notebook in a fresh Google Colab/Linux runtime with:

```python
STUDY_MODE = "PILOT"
```

Execute **Run all** from the first cell.

The PILOT workflow must:

1. install and validate the required scientific and HEIR/OpenFHE dependencies;
2. verify the pinned HEIR source revision and OpenFHE integration;
3. reconstruct and verify the fixed development/locked-test allocation without using locked-test outcomes;
4. train and export the plaintext affine model using development data only;
5. execute both pre-specified CKKS configurations on the complete calibration set;
6. execute the calibration-derived boundary-stress diagnostic;
7. verify successful execution of both frozen CKKS configurations;
8. freeze the complete confirmatory protocol and scientific state;
9. generate and print the confirmatory-freeze SHA-256 digest;
10. export the complete reproducibility bundle.

The printed freeze SHA-256 must be preserved independently from the exported bundle.

### 2. CONFIRMATORY / locked test

Confirmatory evaluation must be performed in a fresh runtime using the complete frozen PILOT bundle.

Configure:

```python
STUDY_MODE = "CONFIRMATORY"
RESTORE_BUNDLE_ZIP = "/content/<pilot-bundle>.zip"
EXPECTED_CONFIRMATORY_FREEZE_SHA256 = "<independently-preserved-digest>"
```

Execute the notebook from the first cell without modifying the frozen scientific analysis.

Before accessing confirmatory outcomes, the workflow must verify:

* the preserved freeze SHA-256;
* the scientific-code integrity manifest;
* the frozen artifact manifest;
* the fixed split provenance;
* the model and operational policy;
* both CKKS configurations;
* relevant dependencies and runtime libraries;
* the environment compatibility contract;
* the pre-specified statistical procedures.

The confirmatory run must terminate if a required frozen scientific or integrity condition is not satisfied.

After locked-test access, no model retraining, configuration selection, boundary tuning, plausibility-threshold tuning, operational-policy tuning, or confirmatory-analysis modification is permitted.

## Interpretation guardrails

The following constraints apply to interpre
