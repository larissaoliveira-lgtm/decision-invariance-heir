# Decision Invariance under Homomorphic Transformation

Open-science repository for studying whether a real CKKS pipeline compiled with HEIR and executed with OpenFHE preserves model outputs and downstream operational decisions under a fixed three-action policy.

> **Current protocol version:** v1.3.0 (hardened, provenance-preserving)
>
> **Current phase:** protocol/PILOT preparation. Confirmatory claims must only be produced after the frozen PILOT bundle has been restored in a fresh runtime and the confirmatory gate has passed.

## Research question

Under a fixed operational policy, how robustly does a real HEIR/OpenFHE CKKS transformation preserve model outputs and downstream decisions, and which part of any discrepancy is attributable to float32 model export versus homomorphic evaluation?

## Pre-specified hypotheses

- **H1 — strict coexistence flag:** binary predictions are exactly preserved while at least one downstream operational action changes. Aggregate binary-fidelity metrics are reported separately; no application-independent non-inferiority margin is imposed.
- **H2 — boundary concentration:** when operational disagreements occur, they are more concentrated near operational decision boundaries.
- **H3 — diagnostic ranking:** when disagreements occur, error normalized by operational margin ranks disagreement more strongly than raw error. H3 is diagnostic rather than a separate reject/accept inferential family.
- **H4 — CKKS configuration comparison:** two pre-specified CKKS configurations differ in incremental HE numerical error after removing the common float32-export component.

## Evidence labels

The repository keeps evidence classes separate:

- `SIMULATED_METHOD_VALIDATION` — method-development evidence only.
- `HEIR_REAL_HE_PILOT_CALIBRATION` — real-HE calibration/feasibility evidence only.
- `HEIR_BOUNDARY_STRESS_DIAGNOSTIC` — mechanistic stress evidence only; **not** a population event-rate estimate.
- `HEIR_REAL_HE_CONFIRMATORY` — the only evidence class supporting locked-test confirmatory claims.

## Study design

The protocol preserves the original v1.0 split of the scikit-learn Wisconsin Breast Cancer benchmark:

- training: 341 cases;
- calibration: 114 cases;
- locked test: 114 cases;
- split seeds: 42 / 43.

The locked test is a provenance-preserving held-out subset, **not an external validation cohort**. The protocol verifies the split identity and locked-test IDs with pre-recorded SHA-256 digests before analysis.

Two CKKS configurations are pre-specified and frozen together:

1. `ckks_reference_f55_s45`
2. `ckks_reduced_f50_s40`

No configuration may be selected after observing locked-test outcomes.

## Robustness analysis

The v1.3 protocol includes:

- decomposition of `float64 -> float32 -> HE` error;
- exact Clopper-Pearson confidence intervals and one-sided upper bounds for rare disagreement events;
- paired bootstrap analysis;
- paired sign-flip randomization test for H4;
- Holm adjustment for the two configuration-specific H2 tests;
- sensitivity analyses for near-boundary definitions and operational policy thresholds;
- calibration-derived boundary stress cases;
- plausibility diagnostics for synthetic stress points using marginal support, Ledoit-Wolf shrinkage Mahalanobis distance, and k-nearest-neighbor distance;
- repeat-level numerical stability diagnostics;
- cryptographic/runtime provenance and artifact hashing;
- transitive code fingerprinting and a scientific-code bundle digest.

## Repository layout

```text
.
├── notebooks/
│   └── OpenScience_HEIR_Decision_Invariance_v1_3_hardened.ipynb
├── protocol/       # frozen protocol and freeze digest after PILOT
├── environment/    # resolved confirmatory requirements after PILOT
├── results/        # generated PILOT/CONFIRMATORY outputs
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
├── CHANGELOG.md
└── .gitignore
```

## Reproduction workflow

### 1. PILOT / calibration

Run the notebook in a fresh Google Colab/Linux runtime with:

```python
STUDY_MODE = "PILOT"
```

Execute **Run all** from the first cell. The notebook must:

1. install and validate the scientific/HEIR dependencies;
2. verify the pinned HEIR source revision and OpenFHE integration;
3. reproduce and verify the original split;
4. train/export the plaintext affine model;
5. execute both pre-specified HE configurations on the complete calibration set;
6. execute the boundary stress diagnostic;
7. register both successful configurations;
8. freeze the complete confirmatory protocol;
9. print a SHA-256 digest for the freeze;
10. export the complete open-science bundle.

Preserve the printed freeze SHA-256 **outside the bundle**.

### 2. CONFIRMATORY / locked test

Use a fresh runtime. Restore the exact PILOT bundle and set:

```python
STUDY_MODE = "CONFIRMATORY"
RESTORE_BUNDLE_ZIP = "/content/<pilot-bundle>.zip"
EXPECTED_CONFIRMATORY_FREEZE_SHA256 = "<independently-preserved-digest>"
```

Run from the first cell. The notebook must reject the run if the scientific state, dependencies, critical code, architecture contract, frozen artifacts, or freeze digest differ from the PILOT record.

## Data

This repository does **not** need to redistribute the Wisconsin Breast Cancer dataset. The notebook loads the dataset through `sklearn.datasets.load_breast_cancer` and records dataset/split provenance. Generated split indices, protocol records, stress-set artifacts, and result tables are frozen and hashed by the study workflow.

## Scope and limitations

The confirmatory family is intentionally narrow:

- one public benchmark dataset;
- one affine logistic-regression workload;
- one three-action operational policy family;
- two pre-specified CKKS configurations;
- HEIR/OpenFHE as the real HE toolchain.

External replication on additional datasets and more complex/nonlinear encrypted workloads is required before making broad claims about ML decision invariance under CKKS.

## Integrity principle

Do not edit a frozen PILOT bundle in place. Any protocol or implementation change after freezing must create a new study version and a new PILOT/freeze. Preserve prior releases as provenance.

## Citation

Citation metadata are provided in [`CITATION.cff`](CITATION.cff). A DOI can be added after the repository is archived through a service such as Zenodo.

## License

**License selection is pending institutional approval.** The repository should not be described as open source until an explicit code license is selected. See `docs/open-science-checklist.md`.

## Author

Larissa de Oliveira Figueira — FACTI
