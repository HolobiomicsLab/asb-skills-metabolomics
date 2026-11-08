---
name: correlated-feature-removal-thresholding
description: Use when a continuous molecular descriptor matrix (e.g., alvaDesc descriptors
  for SMRT molecules) contains highly correlated features that could destabilize downstream
  regressors, after zero-variance features have already been dropped.
when_to_use_negative:
- Feature columns are binary fingerprints (MACCS166/ECFP/PFP) — the paper treats these
  as Bernoulli variables and removes low-variance bits via the p(1-p) variance rule
  instead of correlation thresholding.
- Input is already a cleaned feature matrix with no zero-variance or >0.9-correlated
  pairs — re-applying would be a no-op.
- Correlated features carry interpretive value for the analysis (e.g., studying descriptor
  relationships) rather than serving as regressor inputs.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implements descriptor standardization, median imputation, zero-variance removal,
    and >0.9 correlation-based feature elimination in the CMM-RT preprocessing pipeline
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Source of the 5,666 molecular descriptors being filtered; failures to generate
    a descriptor trigger the median imputation that precedes correlation removal
  repo: https://www.alvascience.com/alvadesc/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:880448f89e6a8f7d4b3a902badd14e6dfca6a98b8d71a94c81ce8ea5b8f72be8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Correlated feature removal via fixed correlation threshold

## Summary

Preprocessing skill that eliminates descriptor features whose pairwise correlation exceeds a fixed 0.9 threshold, applied after standardization and median imputation of alvaDesc-generated molecular descriptors for retention time (RT) prediction regressors.

## When to use

Use when a continuous molecular descriptor matrix (e.g., alvaDesc descriptors for SMRT molecules) contains highly correlated features that could destabilize downstream regressors, after zero-variance features have already been dropped.

## When NOT to use

- Feature columns are binary fingerprints (MACCS166/ECFP/PFP) — the paper treats these as Bernoulli variables and removes low-variance bits via the p(1-p) variance rule instead of correlation thresholding.
- Input is already a cleaned feature matrix with no zero-variance or >0.9-correlated pairs — re-applying would be a no-op.
- Correlated features carry interpretive value for the analysis (e.g., studying descriptor relationships) rather than serving as regressor inputs.

## Inputs

- alvaDesc-generated descriptor matrix (5,666 descriptors) for the 80,038 SMRT molecules, standardized with median-imputed missing values and binary missing-indicator columns
- Descriptor/fingerprint feature matrix with zero-variance features already removed

## Outputs

- cleaned_feature_matrix.parquet — descriptor matrix with >0.9-correlated features eliminated, one row per SMRT molecule (80,038 rows)
- List/index of removed highly correlated descriptor features

## How to apply

In Python, first standardize the alvaDesc descriptors and median-impute any descriptor alvaDesc failed to generate, appending a binary missing-value indicator column for each imputed feature. Remove features with zero variance, then compute pairwise feature correlations and eliminate one feature of any pair with correlation above the fixed 0.9 threshold (the threshold is fixed and untuned in the CMM-RT paper, not optimized via cross-validation). This redundancy removal is applied only to continuous descriptor columns; fingerprint bits (MACCS166, ECFP, PFP) are handled separately as Bernoulli variables using a variance threshold Var[X] = p(1-p) with p tuned above 0.9. Save the cleaned matrix as a parquet file (cleaned_feature_matrix.parquet) preserving one row per molecule — here, the 80,038 SMRT molecules.

## Related tools

- **Python** (Implements descriptor standardization, median imputation, zero-variance removal, and >0.9 correlation-based feature elimination in the CMM-RT preprocessing pipeline) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Source of the 5,666 molecular descriptors being filtered; failures to generate a descriptor trigger the median imputation that precedes correlation removal) — https://www.alvascience.com/alvadesc/

## Evaluation signals

- Output parquet file exists with exactly 80,038 rows (one per SMRT molecule).
- No remaining zero-variance features and no remaining feature pairs with correlation above the 0.9 threshold in the cleaned matrix.
- Each median-imputed descriptor has an accompanying binary missing-value indicator column preserved through filtering.
- The 0.9 correlation cutoff was applied as a fixed, untuned value (not selected by hyperparameter search), matching the paper's protocol.
- Fingerprint columns were excluded from correlation-based removal and instead filtered by the Bernoulli variance criterion Var[X] = p(1-p).

## Limitations

- The 0.9 correlation threshold is fixed and untuned; a different threshold might be preferable for other feature sets or models.
- Removal of correlated features discards potentially informative redundancy without a principled choice of which member of each correlated pair to keep.
- Depends on upstream median imputation: descriptors alvaDesc could not generate must be imputed (with missing-indicator columns) before correlation is computed.

## Evidence

- [methods] Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [readme] 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with"
