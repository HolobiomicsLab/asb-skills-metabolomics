---
name: fingerprint-variance-thresholding
description: Use when preprocessing alvaDesc fingerprint bit columns (MACCS166, Extended
  Connectivity, Path Fingerprints) for the SMRT RT-prediction models and low-prevalence,
  near-constant bits must be dropped before regressor training.
when_to_use_negative:
- The input is an already-cleaned feature table with no low-variance binary columns
  left to remove
- The columns are continuous molecular descriptors rather than binary bits — these
  require the zero-variance / >0.9-correlation filtering route instead of the Bernoulli
  p(1-p) threshold
- Fingerprints were not generated with alvaDesc (e.g. RDKit fingerprints from train_with_rdkit.ipynb),
  where bit-prevalence properties may differ and the tuned p > 0.9 threshold was not
  established
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: alvaDesc
  role: Generates the 2,214 MACCS166, Extended Connectivity and Path fingerprint bits
    whose low-variance columns are thresholded
  repo: https://www.alvascience.com/alvadesc/
- name: Python
  role: Implements the variance thresholding within the cmmrt preprocessing pipeline
    that produces cleaned_feature_matrix.parquet
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:9f539ec27ca08cb5d7a2e541a1c5fd6e5c28b6246411f807d33535929fac07b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - alvaDesc
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Fingerprint variance thresholding

## Summary

Removes low-variance binary fingerprint bits from an alvaDesc-generated fingerprint matrix (MACCS166, ECFP, PFP) before training retention time prediction regressors, treating each bit as a Bernoulli variable whose variance Var[X] = p(1-p) must exceed a tuned threshold with p above 0.9.

## When to use

Use when preprocessing alvaDesc fingerprint bit columns (MACCS166, Extended Connectivity, Path Fingerprints) for the SMRT RT-prediction models and low-prevalence, near-constant bits must be dropped before regressor training.

## When NOT to use

- The input is an already-cleaned feature table with no low-variance binary columns left to remove
- The columns are continuous molecular descriptors rather than binary bits — these require the zero-variance / >0.9-correlation filtering route instead of the Bernoulli p(1-p) threshold
- Fingerprints were not generated with alvaDesc (e.g. RDKit fingerprints from train_with_rdkit.ipynb), where bit-prevalence properties may differ and the tuned p > 0.9 threshold was not established

## Inputs

- alvaDesc-generated descriptor and fingerprint matrix for the 80,038 SMRT molecules (ECFP, MACCSFP, PFP bits plus 5,666 molecular descriptors)
- Binary fingerprint bit columns to be treated as Bernoulli variables

## Outputs

- Cleaned feature matrix (cleaned_feature_matrix.parquet) with one row per SMRT molecule (80,038 rows) and low-variance fingerprint bits removed
- Set of retained fingerprint bits whose Var[X] = p(1-p) exceeds the tuned threshold (p > 0.9)

## How to apply

Load the descriptor and fingerprint matrix for the 80,038 SMRT molecules and treat each fingerprint column as a Bernoulli variable with bit frequency p, so its variance is Var[X] = p(1-p). Drop fingerprint bits whose variance falls below the threshold, tuning p above 0.9 (unlike the descriptor-side correlation cutoff of 0.9, which is fixed and untuned). Perform this step alongside the descriptor preprocessing: standardization, median imputation of descriptors alvaDesc could not generate with binary missing-value indicator columns, removal of zero-variance features, and elimination of correlated features above the 0.9 threshold. Save the cleaned feature matrix as cleaned_feature_matrix.parquet with one row per SMRT molecule so it can feed the RT regressors (XGBoost, LightGBM, CatBoost, DNN).

## Related tools

- **alvaDesc** (Generates the 2,214 MACCS166, Extended Connectivity and Path fingerprint bits whose low-variance columns are thresholded) — https://www.alvascience.com/alvadesc/
- **Python** (Implements the variance thresholding within the cmmrt preprocessing pipeline that produces cleaned_feature_matrix.parquet) — https://github.com/constantino-garcia/cmmrt

## Evaluation signals

- Output cleaned_feature_matrix.parquet exists with exactly 80,038 rows, one per SMRT molecule
- No remaining zero-variance feature columns and no correlated feature pairs above the 0.9 threshold
- Every retained fingerprint bit has Bernoulli variance Var[X] = p(1-p) above the threshold, with the tuned p value above 0.9
- Downstream nested cross-validation of the RT regressors (results/rt/rt_cv.csv) runs without errors on the cleaned matrix
- Dropping the filtered bits does not change the number of SMRT molecules, only the number of feature columns

## Limitations

- The bit-prevalence threshold p is a tuned hyperparameter (tuned above 0.9), so an overly aggressive value can discard rare but informative fingerprint bits
- The Bernoulli Var[X] = p(1-p) rationale only applies to binary fingerprint columns; applying it to continuous descriptors is invalid
- The threshold interacts with the descriptor-side >0.9 correlation filter; both must be applied consistently to reproduce the paper's feature set

## Evidence

- [methods] Removal of zero-variance features: "Features with 0 variance were removed."
- [readme] Fingerprints generated with alvaDesc: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software."
- [methods] Descriptor imputation context: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Correlated feature removal: [evidence span withheld — anchor status: not_found]
