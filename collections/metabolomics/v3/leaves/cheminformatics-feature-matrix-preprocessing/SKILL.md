---
name: cheminformatics-feature-matrix-preprocessing
description: Use when a descriptor/fingerprint feature matrix (e.g., alvaDesc output
  on SMRT molecules) must be reduced before training RT prediction regressors and
  pairwise feature correlation may exceed 0.9.
when_to_use_negative:
- Input is already a reduced feature table or feature selection has already been performed.
- You intend to tune the correlation cutoff as a hyperparameter — the paper explicitly
  keeps the 0.9 threshold untuned.
- The task is RT projection between chromatographic methods (Bayesian GP meta-learning
  on PredRet) rather than feature preprocessing for regressor training.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implements the correlation-based feature elimination and writes the reduced
    feature matrix
- name: cmmrt
  role: Package providing the RT preprocessing/training pipeline in which this filtering
    step runs
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: 'Generates the descriptors (5,666) and fingerprints (2,214: MACCS166, ECFP,
    PFP) that form the matrix to be filtered'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:aac352aeadea4f0d1b9e1ef43b01e48b89863f6647e2c0cd719507e16bf1636f
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
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cheminformatics-feature-matrix-preprocessing

## Summary

Preprocess molecular feature matrices generated with alvaDesc (5,666 descriptors and 2,214 MACCS166/ECFP/PFP fingerprints) for RT prediction by removing zero-variance features and iteratively eliminating one feature from each pair with pairwise correlation >0.9.

## When to use

Use when a descriptor/fingerprint feature matrix (e.g., alvaDesc output on SMRT molecules) must be reduced before training RT prediction regressors and pairwise feature correlation may exceed 0.9.

## When NOT to use

- Input is already a reduced feature table or feature selection has already been performed.
- You intend to tune the correlation cutoff as a hyperparameter — the paper explicitly keeps the 0.9 threshold untuned.
- The task is RT projection between chromatographic methods (Bayesian GP meta-learning on PredRet) rather than feature preprocessing for regressor training.

## Inputs

- SMRT feature matrix generated with alvaDesc (descriptors only, fingerprints only, or descriptors+fingerprints)
- RT labels for each molecule
- Molecule identifiers (e.g., PubChem IDs / InChI from the SMRT CSV input)

## Outputs

- reduced_feature_matrix.csv — filtered feature matrix with molecule identifiers and RT targets preserved

## How to apply

Load the SMRT feature matrix generated with alvaDesc — descriptors, fingerprints, or both combined — together with the RT labels. First drop features with zero variance. Then compute the pairwise feature correlation matrix and iteratively eliminate one feature from each pair whose correlation exceeds the 0.9 threshold; the threshold is deliberately kept untuned because all regressors in the pipeline (XGBoost, LightGBM, CatBoost, DNN) are robust against collinearity. Save the reduced feature matrix, preserving molecule identifiers and RT targets, as reduced_feature_matrix.csv. Note that descriptors were first standardized and median-imputed (with missing indicator) where alvaDesc could not generate them, which should precede correlation computation.

## Related tools

- **Python** (Implements the correlation-based feature elimination and writes the reduced feature matrix)
- **cmmrt** (Package providing the RT preprocessing/training pipeline in which this filtering step runs) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the descriptors (5,666) and fingerprints (2,214: MACCS166, ECFP, PFP) that form the matrix to be filtered)

## Evaluation signals

- Recompute the pairwise correlation matrix on the output: no remaining feature pair should exceed 0.9.
- No feature with zero variance remains in the reduced matrix.
- Row count is unchanged and molecule identifiers and RT targets are preserved alongside the features.
- Output file matches the reduced_feature_matrix.csv schema (identifiers + retained features + RT targets).
- Downstream regressors (XGBoost, LightGBM, CatBoost, DNN) train on the reduced matrix without collinearity-related failures.

## Limitations

- The 0.9 correlation threshold is kept untuned by design; the authors note all regressors are robust against collinearity, so the cutoff was not optimized.
- Iterative elimination of one feature from each correlated pair is order-dependent; which member of a pair survives depends on elimination order.
- Descriptors that alvaDesc cannot generate are median-imputed with a missing indicator, which can affect the computed correlation matrix.

## Evidence

- [methods] Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [methods] Features with 0 variance were removed.: "Features with 0 variance were removed."
- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [readme] 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated"
