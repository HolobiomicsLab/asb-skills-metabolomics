---
name: random-forest-variable-importance-ranking
description: Use when a preprocessed, imputed LC-MS MetaboSet feature table with a
  class label (e.g. group column) is available and you need supervised, model-based
  identification of the most interesting molecular features rather than univariate
  feature-wise tests.
when_to_use_negative:
- The data are raw LC-MS files that have not yet been peak-picked and preprocessed
  into a MetaboSet feature table — notame expects preprocessed LC-MS data.
- The dataset still contains flagged low-quality features or missing values that have
  not been imputed — the ranking must run on the unflagged, imputed features.
- The goal is an unsupervised exploratory overview (e.g. PCA/visual QC) rather than
  supervised feature selection against a class label.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: notame
  role: Provides the MetaboSet data container and preprocessing/QC helpers (e.g. importance_rf
    analogue) that hold the imputed LC-MS feature matrix used as input to MUVR variable
    selection
  repo: https://github.com/antonvsdata/notame
- name: MUVR
  role: R package that runs repeated double cross-validated random-forest and PLS(-DA)
    models and returns 'min', 'mid' and 'max' variable-selection sets with importance
    ranks
- name: missForest
  role: Recommended random-forest imputation strategy that fills missing values in
    the MetaboSet before supervised modelling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:5309f60c007dd7c5214840a4ca8fea1b7f067634325881a3153b1f4512cc3493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - MUVR
  - R
  - missForest
  - notame
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# random-forest-variable-importance-ranking

## Summary

Applies the MUVR package's repeated double cross-validated random-forest (and PLS) models to rank molecular features by importance, producing minimal-optimal ('min'), 'mid' and all-relevant ('max') variable-selection sets for non-targeted LC-MS metabolomics data.

## When to use

Use when a preprocessed, imputed LC-MS MetaboSet feature table with a class label (e.g. group column) is available and you need supervised, model-based identification of the most interesting molecular features rather than univariate feature-wise tests.

## When NOT to use

- The data are raw LC-MS files that have not yet been peak-picked and preprocessed into a MetaboSet feature table — notame expects preprocessed LC-MS data.
- The dataset still contains flagged low-quality features or missing values that have not been imputed — the ranking must run on the unflagged, imputed features.
- The goal is an unsupervised exploratory overview (e.g. PCA/visual QC) rather than supervised feature selection against a class label.

## Inputs

- Preprocessed, imputed MetaboSet object (full_data.RData) from the notame LC-MS preprocessing pipeline
- Class label column (e.g. group) assigning each sample to a biological condition
- MUVR model settings: nRep, nOuter, nInner, varRatio and method (RF and PLS)

## Outputs

- ranked_features_muvr.csv: tidy table with columns model, variable-selection set ('min'/'mid'/'max'), Feature_ID and importance rank
- Importance scores per feature: VIP scores for PLS(-DA) and permutation error-rate increase for random forest

## How to apply

Load the preprocessed, imputed MetaboSet object (full_data.RData produced by the notame preprocessing pipeline) in R with the notame package. Call muvr_analysis on the unflagged features using the class label y (the group column) and MUVR's repeated double cross-validation parameters (nRep, nOuter, nInner, varRatio), specifying the method so both random-forest and PLS(-DA) models are fitted. Extract the three variable-selection model sets MUVR returns per analysis — the minimal-optimal ('min'), 'mid' and all-relevant ('max') sets — which represent increasingly inclusive feature subsets. Rank features within each model and selection set by model-specific importance: VIP scores for PLS(-DA) and permutation error-rate increase (analogous to notame's importance_rf helper) for random forest. Assemble a tidy table with model, variable-selection set, Feature_ID and importance rank per row and write it out (e.g. ranked_features_muvr.csv).

## Related tools

- **notame** (Provides the MetaboSet data container and preprocessing/QC helpers (e.g. importance_rf analogue) that hold the imputed LC-MS feature matrix used as input to MUVR variable selection) — https://github.com/antonvsdata/notame
- **MUVR** (R package that runs repeated double cross-validated random-forest and PLS(-DA) models and returns 'min', 'mid' and 'max' variable-selection sets with importance ranks)
- **missForest** (Recommended random-forest imputation strategy that fills missing values in the MetaboSet before supervised modelling)

## Examples

```
muvr_analysis(set = full_data, y = 'group', method = 'RF', nRep = 5, nOuter = 6, varRatio = 0.75)
```

## Evaluation signals

- ranked_features_muvr.csv exists and parses as a table with model / set / Feature_ID / importance-rank columns
- The output contains ranked entries for BOTH random forest and PLS(-DA) models
- All three variable-selection sets ('min', 'mid' and 'max') are represented in the rankings
- Only unflagged, imputed features appear in the rankings (no low-quality or NA features)
- Every Feature_ID in the output maps to a feature present in the input MetaboSet feature matrix

## Limitations

- The notame package API is still quite experimental and breaking changes are possible, so function signatures (e.g. muvr_analysis arguments) may shift between versions.
- The active development has moved from antonvsdata/notame to hanhineva-lab/notame, so the archived repository is retained only for historical reference.
- Results depend on MUVR's repeated double cross-validation settings (nRep, nOuter, nInner, varRatio); different settings can change which features fall into the 'min', 'mid' and 'max' sets.
- Random-forest importance relies on permutation error-rate increase, which can be unstable on small sample sizes common in metabolomics.

## Evidence

- [results] MUVR variable selection with min/mid/max sets: [evidence span withheld — anchor status: not_found]
- [readme] notame stores data in a custom object: [evidence span withheld — anchor status: external_doc]
- [readme] notame offers feature-wise and multivariate statistical analyses: [evidence span withheld — anchor status: external_doc]
- [readme] notame expects preprocessed LC-MS data: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics."
- [readme] notame API instability caveat: "The package API is still quite experimental, and breaking changes are possible"
