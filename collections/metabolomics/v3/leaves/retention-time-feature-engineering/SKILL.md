---
name: retention-time-feature-engineering
description: Use when building molecular feature inputs (descriptors and/or fingerprints)
  for RT regressors on compounds with SMILES/SDF structures and RT labels, and you
  must handle missing descriptors and collinear features.
when_to_use_negative:
- Input is already a clean, reduced feature matrix with no zero-variance or highly
  correlated features — re-running the filter would only remove features unnecessarily
- Structures are not expressible as SMILES/mol/SDF/mol2/hin or alvaDesc is unavailable
  under license — use RDKit fingerprints instead (see train_with_rdkit.ipynb)
- The downstream task is RT projection between chromatographic methods, not regressor
  training — feature engineering is not the bottleneck there
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 MACCS166/Extended Connectivity/Path
    fingerprints used to build the feature matrix (under license)
  repo: https://www.alvascience.com/alvadesc/
- name: cmmrt
  role: Python package providing build_data.py / build_data_smrt.py / build_data_cmm.py
    utilities (generate_vector_fingerprints, generate_vector_fps_descs) for feature
    generation and the correlation-filter pipeline
  repo: https://github.com/constantino-garcia/cmmrt
- name: RDKit
  role: Alternative fingerprint source for training the DNN when alvaDesc is not available
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:cd6ffc1b38e0983051f27fbdf9cfa2c9c5c8895dfe25d09c62765a4fb3a45406
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# retention-time-feature-engineering

## Summary

Generates and cleans molecular feature matrices (5,666 alvaDesc descriptors and 2,214 ECFP/MACCSFP/PFP fingerprints) for retention time (RT) prediction on the SMRT dataset, applying median imputation, zero-variance removal, and correlation-based feature elimination.

## When to use

Use when building molecular feature inputs (descriptors and/or fingerprints) for RT regressors on compounds with SMILES/SDF structures and RT labels, and you must handle missing descriptors and collinear features.

## When NOT to use

- Input is already a clean, reduced feature matrix with no zero-variance or highly correlated features — re-running the filter would only remove features unnecessarily
- Structures are not expressible as SMILES/mol/SDF/mol2/hin or alvaDesc is unavailable under license — use RDKit fingerprints instead (see train_with_rdkit.ipynb)
- The downstream task is RT projection between chromatographic methods, not regressor training — feature engineering is not the bottleneck there

## Inputs

- CSV file of compounds with pubchem id plus inchi or SMILES structures (e.g. SMRT dataset)
- Structure files in SMILES, mol, SDF, mol2 or hin format
- RT labels (e.g. 80,038 experimental RTs from the METLIN SMRT dataset)
- alvaDesc-generated descriptor and fingerprint vectors (ECFP, MACCSFP, PFP)

## Outputs

- reduced_feature_matrix.csv containing the correlation-filtered feature matrix with molecule identifiers and RT targets
- Concatenated fingerprint strings (ECFP+MACCSFP+PFP) per compound
- Preprocessed/standardized descriptor table with median-imputed missing values

## How to apply

Compute features with alvaDesc: either fingerprints (generate_vector_fingerprints, concatenating ECFP, MACCSFP and PFP values) or descriptors+fingerprints (generate_vector_fps_descs) from SMILES, mol, SDF, mol2 or hin inputs. Standardize descriptors and apply median imputation where alvaDesc cannot generate a descriptor value. Drop zero-variance features, then compute the pairwise feature correlation matrix and iteratively eliminate one feature from each pair whose correlation exceeds 0.9 — this threshold is deliberately left untuned because the downstream regressors (XGBoost, LightGBM, CatBoost, DNN blender) are robust against collinearity. Optionally apply a quantile transformation to the RTs before fitting. Save the reduced matrix, preserving molecule identifiers and RT targets, as reduced_feature_matrix.csv.

## Related tools

- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 MACCS166/Extended Connectivity/Path fingerprints used to build the feature matrix (under license)) — https://www.alvascience.com/alvadesc/
- **cmmrt** (Python package providing build_data.py / build_data_smrt.py / build_data_cmm.py utilities (generate_vector_fingerprints, generate_vector_fps_descs) for feature generation and the correlation-filter pipeline) — https://github.com/constantino-garcia/cmmrt
- **RDKit** (Alternative fingerprint source for training the DNN when alvaDesc is not available)

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models --random_state 42
```

## Evaluation signals

- No pairwise feature correlation greater than 0.9 remains in the output matrix
- All zero-variance columns are absent from reduced_feature_matrix.csv
- Missing descriptor entries are replaced by column medians with an accompanying missing indicator
- Molecule identifiers and RT targets are preserved alongside the reduced feature columns
- Pre-filter feature count matches the expected ~5,666 descriptors plus 2,214 fingerprints (ECFP, MACCSFP, PFP) from alvaDesc

## Limitations

- alvaDesc may fail to generate some descriptors, requiring median imputation with a missing indicator; alvaDesc itself is licensed software
- The 0.9 correlation threshold is kept untuned by design — it is not optimized because all regressors are assumed robust against collinearity
- README results suggest fingerprints tend to perform better than descriptors for RT prediction, so descriptor-heavy features alone may underperform

## Evidence

- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [methods] quantile transformation was applied to RTs before fitting.: "quantile transformation was applied to RTs before fitting."
- [readme] 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software.: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with"
