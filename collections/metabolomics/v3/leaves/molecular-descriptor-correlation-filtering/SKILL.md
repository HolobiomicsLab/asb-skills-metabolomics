---
name: molecular-descriptor-correlation-filtering
description: Use when a high-dimensional molecular feature matrix (alvaDesc descriptors
  and/or MACCS166, ECFP, PFP fingerprints) must be reduced before fitting ML regressors
  such as XGBoost, LightGBM, CatBoost, or a DNN.
when_to_use_negative:
- Input is already a reduced feature matrix or a raw fingerprint-only table where
  per-bit features are intentionally kept despite collinearity (fingerprints tend
  to perform better even without aggressive pruning)
- The downstream model requires all original features (e.g., reproducing the exact
  5,666 descriptor / 2,214 fingerprint alvaDesc output for comparison)
- Data are RT labels or projection targets rather than a feature matrix — the filter
  applies only to predictor columns
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166,
    ECFP, PFP) that form the matrix the correlation filter is applied to
- name: Python
  role: Implements the zero-variance removal and pairwise-correlation (>0.9) elimination
    within the cmmrt preprocessing pipeline
  repo: https://github.com/constantino-garcia/cmmrt
- name: cmmrt
  role: Package providing the preprocessing pipeline (build_data.py / train_model.py)
    that produces and consumes the reduced feature matrix
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:075de852423179d5547641723634257b89b584c429370b908fd3fb8340187415
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# molecular-descriptor-correlation-filtering

## Summary

Removes zero-variance and pairwise-correlated features (correlation > 0.9) from alvaDesc-generated molecular descriptors and fingerprints (ECFP, MACCSFP, PFP) before training RT prediction regressors on the SMRT dataset.

## When to use

Use when a high-dimensional molecular feature matrix (alvaDesc descriptors and/or MACCS166, ECFP, PFP fingerprints) must be reduced before fitting ML regressors such as XGBoost, LightGBM, CatBoost, or a DNN.

## When NOT to use

- Input is already a reduced feature matrix or a raw fingerprint-only table where per-bit features are intentionally kept despite collinearity (fingerprints tend to perform better even without aggressive pruning)
- The downstream model requires all original features (e.g., reproducing the exact 5,666 descriptor / 2,214 fingerprint alvaDesc output for comparison)
- Data are RT labels or projection targets rather than a feature matrix — the filter applies only to predictor columns

## Inputs

- Feature matrix of alvaDesc molecular descriptors and/or fingerprints (ECFP, MACCSFP, PFP) with molecule identifiers (e.g., PubChem ID) as rows
- Retention time (RT) labels from the SMRT dataset (80,038 experimental RTs)
- CSV input file with PubChem ID and SMILES/InChI used to build the alvaDesc feature file

## Outputs

- reduced_feature_matrix.csv — zero-variance and correlated features (>0.9) removed, molecule identifiers and RT targets preserved

## How to apply

Load the SMRT feature matrix generated with alvaDesc (5,666 descriptors and 2,214 fingerprints, optionally combined) together with the RT labels. First drop features with zero variance. Then compute the pairwise feature correlation matrix and iteratively eliminate one feature from each pair whose correlation exceeds the 0.9 threshold, keeping the threshold untuned because all regressors (GBMs and the regularized DNN) are robust against collinearity. Before filtering, descriptors should have been standardized and median-imputed where alvaDesc could not compute them. Save the reduced feature matrix, preserving molecule identifiers (e.g., PubChem IDs) and RT targets, as reduced_feature_matrix.csv for downstream regressor training and blending.

## Related tools

- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP) that form the matrix the correlation filter is applied to)
- **Python** (Implements the zero-variance removal and pairwise-correlation (>0.9) elimination within the cmmrt preprocessing pipeline) — https://github.com/constantino-garcia/cmmrt
- **cmmrt** (Package providing the preprocessing pipeline (build_data.py / train_model.py) that produces and consumes the reduced feature matrix) — https://github.com/constantino-garcia/cmmrt

## Examples

```
python cmmrt/rt/build_data_cmm.py  # generate ECFP/MACCSFP/PFP fingerprints + descriptors, then drop zero-variance and pairwise-correlated features (>0.9) before train_model.py
```

## Evaluation signals

- Output reduced_feature_matrix.csv contains no feature pairs with Pearson correlation above 0.9
- All zero-variance columns present in the original alvaDesc matrix are absent from the output
- Molecule identifier column (PubChem ID) and RT target column are preserved with the original 80,038-row alignment
- Feature count is reduced relative to the input (from up to 5,666 descriptors + 2,214 fingerprints) and no rows are dropped
- Downstream regressor training (train_predictor with Optuna/Bayesian hyperparameter search) runs without NaNs or shape errors on the reduced matrix

## Limitations

- The 0.9 correlation threshold is kept untuned, so it is not optimized per feature set or chromatographic method
- Median imputation is needed first because alvaDesc may fail to generate some descriptors; filtering on unimputed data would be biased
- The filter is part of a supervised RT-prediction pipeline (SMRT dataset, reversed-phase LCMS); it was not validated for other feature semantics or tasks

## Evidence

- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Features with 0 variance were removed. Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [readme] 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with"
- [readme] generate_vector_fps_descs(aDesc, chemicalStructureFile, fingerprint_types = ("ECFP", "MACCSFP", "PFP"), descriptors = True): "generate_vector_fps_descs(aDesc, chemicalStructureFile, fingerprint_types = ("ECFP", "MACCSFP", "PFP"), descriptors = True)"
