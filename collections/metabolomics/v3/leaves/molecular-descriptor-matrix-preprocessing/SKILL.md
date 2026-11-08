---
name: molecular-descriptor-matrix-preprocessing
description: Use when an alvaDesc descriptor/fingerprint matrix (e.g. 5,666 descriptors
  + 2,214 ECFP/MACCS166/PFP bits for 80,038 SMRT molecules) contains NaNs from failed
  descriptor generation, zero-variance columns, or highly correlated features and
  must be cleaned before fitting RT regressors.
when_to_use_negative:
- The input is already a clean numeric feature table with no missing values — re-imputing
  and dropping features would change the feature space away from the paper's benchmark
  configuration.
- The goal is RT projection between chromatographic methods (PredRet meta-learning),
  where RTs are instead log-transformed and robustly scaled using median and IQR —
  that pipeline operates on targets, not descriptor matrices.
- Fingerprints come from RDKit (e.g. the train_with_rdkit.ipynb workflow) rather than
  alvaDesc, since the missing-value pattern and Bernoulli bit filtering were designed
  for the alvaDesc-generated matrix.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3174
tools:
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 fingerprints (ECFP, MACCSFP,
    PFP) whose NaNs, zero-variance columns, and correlations this preprocessing step
    cleans up
- name: Python (cmmrt package)
  role: Implements the preprocessing pipeline and feature generation utilities (build_data_cmm.py,
    build_data_smrt.py) used to produce the cleaned matrix for the RT regressors
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:29433acbd240c5f514d3f9d1983d2258d4e4bbb9a186e5f696f0a67cd8eeca2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - CatBoost
  - LightGBM
  - Python
  - Python (cmmrt package)
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# molecular-descriptor-matrix-preprocessing

## Summary

Cleans alvaDesc-generated molecular descriptor and fingerprint matrices for SMRT retention-time regressors by standardizing descriptors, median-imputing failed descriptor values with binary missing indicators, and dropping zero-variance, >0.9-correlated, and low-variance Bernoulli fingerprint bits.

## When to use

Use when an alvaDesc descriptor/fingerprint matrix (e.g. 5,666 descriptors + 2,214 ECFP/MACCS166/PFP bits for 80,038 SMRT molecules) contains NaNs from failed descriptor generation, zero-variance columns, or highly correlated features and must be cleaned before fitting RT regressors.

## When NOT to use

- The input is already a clean numeric feature table with no missing values — re-imputing and dropping features would change the feature space away from the paper's benchmark configuration.
- The goal is RT projection between chromatographic methods (PredRet meta-learning), where RTs are instead log-transformed and robustly scaled using median and IQR — that pipeline operates on targets, not descriptor matrices.
- Fingerprints come from RDKit (e.g. the train_with_rdkit.ipynb workflow) rather than alvaDesc, since the missing-value pattern and Bernoulli bit filtering were designed for the alvaDesc-generated matrix.

## Inputs

- alvaDesc descriptor matrix (5,666 molecular descriptors) with NaNs for descriptors alvaDesc failed to generate, one row per molecule (80,038 SMRT records)
- alvaDesc fingerprint matrix (2,214 bits: ECFP, MACCS166, PFP), one row per molecule
- CSV input file containing PubChem id and SMILES/InChI per compound (e.g. build_data_cmm.py / build_data_smrt.py input)
- SDF structure files referenced by PubChem id

## Outputs

- cleaned_feature_matrix.parquet with one row per SMRT molecule (80,038 rows), no zero-variance or >0.9-correlated features
- Standardized descriptor columns with median-imputed values and one binary missing-value indicator column per imputed descriptor
- Fingerprint bit columns with low-variance Bernoulli bits removed (p threshold > 0.9)

## How to apply

Load the alvaDesc-generated descriptor and fingerprint matrix (5,666 descriptors and 2,214 MACCS166, Extended Connectivity, and Path fingerprints for the 80,038 SMRT molecules) and standardize the descriptors, since alvaDesc leaves NaNs where it could not compute a descriptor — median-impute those values and append a binary missing-value indicator column for each imputed feature. Remove features with zero variance, then eliminate highly correlated features using a fixed, untuned Pearson correlation threshold of >0.9. Treat fingerprint columns as Bernoulli variables and remove low-variance bits using the Var[X] = p(1−p) criterion with the p threshold tuned above 0.9. Save the result (e.g. cleaned_feature_matrix.parquet, one row per SMRT molecule); the cleaned matrix then feeds the DNN/GBM regressor training, where a quantile transformation is applied to the RT targets separately. The rationale is that the heavily regularized DNN benchmark (MAE 39.2±1.2 s, MEDAE 17.2±0.9 s) was trained on exactly this preprocessed feature space, so reproducing the paper's numbers requires the same preprocessing.

## Related tools

- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 fingerprints (ECFP, MACCSFP, PFP) whose NaNs, zero-variance columns, and correlations this preprocessing step cleans up)
- **Python (cmmrt package)** (Implements the preprocessing pipeline and feature generation utilities (build_data_cmm.py, build_data_smrt.py) used to produce the cleaned matrix for the RT regressors) — https://github.com/constantino-garcia/cmmrt

## Examples

```
from sklearn.impute import SimpleImputer; X_clean = SimpleImputer(strategy="median", add_indicator=True).fit_transform(X_desc)
```

## Evaluation signals

- Output parquet has exactly one row per SMRT molecule (80,038 rows) and no remaining NaN values in descriptor columns.
- No zero-variance feature columns remain, and no descriptor pair has absolute Pearson correlation >0.9.
- Each median-imputed descriptor is accompanied by its binary missing-value indicator column, and fingerprint bits retained all satisfy the Bernoulli variance criterion Var[X] = p(1−p) with p threshold tuned above 0.9.
- Downstream nested-CV DNN (fingerprints) reproduces MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s on all SMRT molecules, as in Tables 1 and Figs. 5/S3.
- The blender/summary outputs (results/rt/rt_cv.csv) exist and are populated after re-running the pipeline via the Makefile targets.

## Limitations

- The 0.9 correlation threshold is fixed and untuned; no sensitivity analysis of this cutoff is provided, so other thresholds are unsupported by this paper.
- alvaDesc is commercial, licensed software required to regenerate the descriptors/fingerprints; the code cannot build the feature matrix without it.
- For reproducing the paper's results, the repository's `paper` branch must be used because the code continues to be developed for CEU Mass Mediator integration.
- The Bernoulli p threshold for fingerprint bit removal is a tunable hyperparameter (>0.9), so exact bit counts may vary between hyperparameter search trials.

## Evidence

- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Highly correlated features eliminated: [evidence span withheld — anchor status: not_found]
- [methods] Features with 0 variance were removed.: "Features with 0 variance were removed."
- [readme] 5,666 descriptors and 2,214 fingerprints generated with alvaDesc: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software."
- [methods] quantile transformation applied to RTs before fitting: "quantile transformation was applied to RTs before fitting."
