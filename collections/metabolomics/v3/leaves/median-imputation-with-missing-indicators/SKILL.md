---
name: median-imputation-with-missing-indicators
description: Use when a descriptor/fingerprint feature matrix contains NaNs from tool
  failures (e.g., alvaDesc not generating some of the 5,666 descriptors) before fitting
  tree- or neural-network-based retention time regressors.
when_to_use_negative:
- Input is already a clean, complete feature table with no alvaDesc-generated NaNs
  — imputation would add unnecessary columns and mask real data structure
- The target variable (RT values) is the feature being preprocessed — RTs instead
  receive a quantile transformation before fitting, not median imputation
- Fingerprint bits only, treated with their own Bernoulli p(1-p) low-variance rule
  — do not apply descriptor-style standardization/median imputation to them
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: alvaDesc
  role: Source of the 5,666 molecular descriptors and 2,214 fingerprints whose per-molecule
    failures produce the missing values that this imputation step handles
- name: Python
  role: Implements the standardization, median imputation, indicator-column appending,
    and variance/correlation filtering within the cmmrt pipeline
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:e26095d46b5d7f6d4cb3293015f54329ed97b02bee31bb134b11810b7b1500af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# median-imputation-with-missing-indicators

## Summary

Preprocess alvaDesc-generated molecular descriptor matrices for RT-prediction regressors by standardizing features, median-imputing descriptors that alvaDesc failed to generate, and appending a binary missing-value indicator column per imputed feature so imputation is not silently lost.

## When to use

Use when a descriptor/fingerprint feature matrix contains NaNs from tool failures (e.g., alvaDesc not generating some of the 5,666 descriptors) before fitting tree- or neural-network-based retention time regressors.

## When NOT to use

- Input is already a clean, complete feature table with no alvaDesc-generated NaNs — imputation would add unnecessary columns and mask real data structure
- The target variable (RT values) is the feature being preprocessed — RTs instead receive a quantile transformation before fitting, not median imputation
- Fingerprint bits only, treated with their own Bernoulli p(1-p) low-variance rule — do not apply descriptor-style standardization/median imputation to them

## Inputs

- alvaDesc-generated descriptor and fingerprint matrix (5,666 descriptors + 2,214 MACCS166/ECFP/PFP fingerprint bits) for the 80,038 SMRT molecules, containing NaNs where alvaDesc failed to compute a descriptor
- SMILES / SDF / mol structure files used as alvaDesc input

## Outputs

- Cleaned feature matrix (e.g., cleaned_feature_matrix.parquet) with one row per SMRT molecule (80,038 rows), standardized and median-imputed descriptor columns, and appended binary missing-value indicator columns
- Feature set with zero-variance and >0.9-correlated features removed

## How to apply

Load the alvaDesc descriptor and fingerprint matrix (5,666 descriptors plus MACCS166, ECFP and PFP fingerprints for the 80,038 SMRT molecules). Standardize the descriptor features, then impute missing values — those where alvaDesc could not generate a descriptor — using the feature median, and append a binary indicator column for each imputed feature so downstream models can distinguish imputed from observed values. After imputation, drop zero-variance features and remove highly correlated features (correlation >0.9, a fixed untuned threshold). Treat fingerprint bits as Bernoulli variables and remove low-variance bits with the Var[X] = p(1-p) threshold, with p tuned above 0.9. This ordering matters: imputation must precede variance/correlation filtering so that constant or degenerate imputed columns are caught by the later filters.

## Related tools

- **alvaDesc** (Source of the 5,666 molecular descriptors and 2,214 fingerprints whose per-molecule failures produce the missing values that this imputation step handles)
- **Python** (Implements the standardization, median imputation, indicator-column appending, and variance/correlation filtering within the cmmrt pipeline) — https://github.com/constantino-garcia/cmmrt

## Evaluation signals

- Output feature matrix has exactly one row per SMRT molecule (80,038 rows)
- No NaN values remain in the cleaned matrix; every imputed feature is accompanied by its binary missing-value indicator column
- No features with zero variance and no feature pairs with correlation above 0.9 survive in the output
- Fingerprint bits passing the filter satisfy the Bernoulli variance threshold Var[X] = p(1-p) with p above 0.9
- Downstream RT regressors (e.g., the DNN reaching 39.2±1.2 s MAE on SMRT) train without imputation-related errors

## Limitations

- Correlation threshold (>0.9) was fixed and untuned, so borderline correlated features may be dropped or kept inconsistently relative to a tuned cutoff
- Median imputation assumes missing descriptor failures are uninformative; the missingness itself is only preserved via the appended indicator columns
- Imputation applies to descriptors only — alvaDesc fingerprint bits are handled separately with a Bernoulli low-variance rule, and the paper's best model used fingerprints, for which this imputation pathway is less relevant

## Evidence

- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] zero-variance removal: "Features with 0 variance were removed."
- [methods] correlation filtering: [evidence span withheld — anchor status: not_found]
- [readme] alvaDesc feature generation scale: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software."
