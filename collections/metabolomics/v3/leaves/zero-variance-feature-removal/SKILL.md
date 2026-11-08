---
name: zero-variance-feature-removal
description: Use when a molecular descriptor or fingerprint matrix (e.g. alvaDesc
  output for SMRT-like datasets) contains constant features or uninformative fingerprint
  bits that must be removed prior to regression or model training.
when_to_use_negative:
- The input is already a reduced/cleaned feature matrix that has passed variance and
  correlation filtering
- The goal is to tune the >0.9 correlation filter threshold — the article keeps these
  filter parameters fixed and untuned, reasoning that all regressors are robust against
  collinearity
- Fingerprint bits are not Bernoulli variables (the p(1-p) variance criterion applies
  only to binary fingerprint columns)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: alvaDesc
  role: Generates the 5,666 descriptors and 2,214 fingerprints (MACCS166, ECFP, Path
    Fingerprints) whose zero-variance features are removed
- name: Python
  role: Implements the variance filtering and matrix cleanup in the cmmrt package
    pipeline
  repo: https://github.com/constantino-garcia/cmmrt
- name: cmmrt
  role: Package whose RT preprocessing pipeline (build_data / train_model scripts)
    performs the zero-variance feature removal before regressor training
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:eb1db4ebfec5e53d347c63eaf6edadd4be3308b4ab2972b217f44bdd5110481d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# zero-variance-feature-removal

## Summary

A feature-preprocessing step from the CMM-RT retention time prediction pipeline that drops constant (zero-variance) columns from the alvaDesc descriptor/fingerprint matrix (5,666 descriptors and 2,214 MACCS166/ECFP/Path fingerprints) before regressor training, and applies an analogous p(1-p) variance threshold to fingerprint bits.

## When to use

Use when a molecular descriptor or fingerprint matrix (e.g. alvaDesc output for SMRT-like datasets) contains constant features or uninformative fingerprint bits that must be removed prior to regression or model training.

## When NOT to use

- The input is already a reduced/cleaned feature matrix that has passed variance and correlation filtering
- The goal is to tune the >0.9 correlation filter threshold — the article keeps these filter parameters fixed and untuned, reasoning that all regressors are robust against collinearity
- Fingerprint bits are not Bernoulli variables (the p(1-p) variance criterion applies only to binary fingerprint columns)

## Inputs

- alvaDesc-generated descriptor and fingerprint matrix (CSV/parquet) with one row per molecule (e.g. 80,038 SMRT rows), including descriptor columns and ECFP/MACCSFP/PFP fingerprint bits
- Retention time labels (RT targets) associated with each molecule

## Outputs

- Cleaned feature matrix (e.g. cleaned_feature_matrix.parquet) with zero-variance features removed, one row per SMRT molecule
- For fingerprints, a reduced bit set where low-variance bits (Var[X] = p(1-p), p tuned above 0.9) have been dropped

## How to apply

Load the alvaDesc-generated descriptor and fingerprint matrix (e.g. the 80,038-molecule SMRT matrix) and, after standardizing and median-imputing descriptors (appending binary missing-value indicators), compute the variance of each feature column and remove any feature with zero variance. For fingerprint columns treated as Bernoulli variables, instead remove low-variance bits using the Var[X] = p(1-p) criterion with the p threshold tuned above 0.9. This step runs before the correlation filter that eliminates one feature from each pair with pairwise correlation >0.9, and the cleaned matrix is saved preserving one row per molecule. The rationale is to strip features that carry no discriminative information before feeding descriptors, fingerprints, or both to RT prediction regressors (XGBoost, LightGBM, CatBoost, and a heavily regularized DNN).

## Related tools

- **alvaDesc** (Generates the 5,666 descriptors and 2,214 fingerprints (MACCS166, ECFP, Path Fingerprints) whose zero-variance features are removed)
- **Python** (Implements the variance filtering and matrix cleanup in the cmmrt package pipeline) — https://github.com/constantino-garcia/cmmrt
- **cmmrt** (Package whose RT preprocessing pipeline (build_data / train_model scripts) performs the zero-variance feature removal before regressor training) — https://github.com/constantino-garcia/cmmrt

## Evaluation signals

- No remaining feature columns with zero variance in the cleaned matrix
- Output file has exactly one row per input molecule (80,038 rows for the full SMRT dataset)
- No remaining zero-variance or >0.9-correlated feature pairs after the downstream correlation filter also runs
- Fingerprint bit counts shrink consistently with a p(1-p) threshold tuned above 0.9 (i.e. only near-constant bits removed)
- Molecule identifiers and RT targets are preserved unchanged alongside the filtered features

## Limitations

- The 0.9 correlation threshold and fingerprint p cutoff are fixed and untuned; the authors argue this is acceptable because all regressors are robust against collinearity, so users should not expect the filter to be optimized
- Zero-variance removal is a precondition of the correlation filter — running the correlation filter on an unfiltered matrix may behave differently from the paper's pipeline
- Descriptors that alvaDesc failed to generate must first be median-imputed (with missing-value indicator columns) before variance filtering, otherwise imputation artifacts can distort feature statistics

## Evidence

- [methods] Zero-variance removal precedes correlation filtering: "Features with 0 variance were removed."
- [methods] Descriptors were standardized and imputed before filtering: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Correlation filter follows as the next preprocessing step: [evidence span withheld — anchor status: not_found]
- [readme] Feature space scale motivating the filter: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software."
