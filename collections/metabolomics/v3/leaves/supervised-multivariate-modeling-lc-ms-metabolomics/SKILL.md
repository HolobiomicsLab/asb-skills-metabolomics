---
name: supervised-multivariate-modeling-lc-ms-metabolomics
description: Use when a preprocessed, imputed LC-MS feature table (e.g. a notame MetaboSet
  loaded from full_data.RData) with a class label such as 'group' needs supervised
  multivariate modeling to rank molecular features.
when_to_use_negative:
- Input is raw LC-MS data that has not yet been preprocessed, drift-corrected, flagged
  and imputed (e.g. not yet passed through notame's mark_nas, correct_drift, flag_quality
  and impute_rf steps) — supervised modeling needs a clean, imputed feature matrix
- The analysis goal is unsupervised exploration (clustering similar molecular features,
  PCA) or per-feature univariate tests rather than supervised feature ranking against
  a class label
- The dataset has no meaningful class label (no 'group' outcome) to predict, since
  both RF and PLS(-DA) here are fitted against y
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: notame
  role: Stores the preprocessed LC-MS feature abundance data in its MetaboSet object
    and supplies the imputed, unflagged features and class labels that feed the supervised
    models
  repo: https://github.com/antonvsdata/notame
- name: MUVR
  role: R package whose muvr_analysis function fits repeated double cross-validated
    RF and PLS(-DA) models and returns 'min', 'mid' and 'max' variable-selection sets
- name: R
  role: Runtime environment in which the notame MetaboSet is loaded and muvr_analysis
    is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:8ae2709aeadce999c7f474bdfae8523587c6c8ad0f16699f350c448298602600
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# supervised-multivariate-modeling-lc-ms-metabolomics

## Summary

Applies the MUVR package's repeated double cross-validated RF and PLS(-DA) models to a preprocessed, imputed notame MetaboSet object to identify the most interesting molecular features, returning the minimal-optimal ('min'), 'mid' and all-relevant ('max') variable-selection sets ranked by model-specific importance.

## When to use

Use when a preprocessed, imputed LC-MS feature table (e.g. a notame MetaboSet loaded from full_data.RData) with a class label such as 'group' needs supervised multivariate modeling to rank molecular features.

## When NOT to use

- Input is raw LC-MS data that has not yet been preprocessed, drift-corrected, flagged and imputed (e.g. not yet passed through notame's mark_nas, correct_drift, flag_quality and impute_rf steps) — supervised modeling needs a clean, imputed feature matrix
- The analysis goal is unsupervised exploration (clustering similar molecular features, PCA) or per-feature univariate tests rather than supervised feature ranking against a class label
- The dataset has no meaningful class label (no 'group' outcome) to predict, since both RF and PLS(-DA) here are fitted against y

## Inputs

- Preprocessed, imputed notame MetaboSet object (e.g. full_data.RData from the notame project-example pipeline)
- Class label / grouping variable (e.g. the 'group' column) for supervised modeling
- List of unflagged molecular features (low-quality features already flagged via notame flag_detection/flag_quality)

## Outputs

- ranked_features_muvr.csv: tidy table with model, variable-selection set ('min'/'mid'/'max'), Feature_ID and importance rank per row
- Per-model feature rankings: VIP scores for PLS(-DA) and permutation error-rate increase for RF

## How to apply

Load the preprocessed, imputed MetaboSet object (full_data.RData from the notame preprocessing pipeline) in R and restrict to unflagged features. Run muvr_analysis using the class label y (the group column) with MUVR's repeated double cross-validation parameters (nRep, nOuter, nInner, varRatio) so that both random forest (RF) and PLS(-DA) models are fitted. Extract the three variable-selection model sets MUVR returns per analysis: the minimal-optimal 'min', the 'mid', and the all-relevant 'max' sets, which represent increasingly inclusive feature subsets. Rank features within each model and selection set by model-specific importance: VIP scores for PLS(-DA) and permutation error-rate increase for random forest. Assemble a tidy table with model, variable-selection set, Feature_ID and importance rank per row, and write it to ranked_features_muvr.csv.

## Related tools

- **notame** (Stores the preprocessed LC-MS feature abundance data in its MetaboSet object and supplies the imputed, unflagged features and class labels that feed the supervised models) — https://github.com/antonvsdata/notame
- **MUVR** (R package whose muvr_analysis function fits repeated double cross-validated RF and PLS(-DA) models and returns 'min', 'mid' and 'max' variable-selection sets)
- **R** (Runtime environment in which the notame MetaboSet is loaded and muvr_analysis is executed)

## Examples

```
muvr_analysis(set, y = "group", method = "RF", nRep = 5, nOuter = 6, nInner = 10, varRatio = 0.5)
```

## Evaluation signals

- ranked_features_muvr.csv exists and parses as a table with model, set, Feature_ID and importance-rank columns
- The table contains ranked entries for both RF and PLS(-DA) across all three ('min', 'mid', 'max') selection sets
- Set sizes follow the expected ordering: 'min' is the smallest (minimal-optimal) subset and 'max' is the most inclusive (all-relevant) subset
- The importance metric used matches the model type: VIP scores for PLS(-DA) rows and permutation error-rate increase for RF rows
- Every Feature_ID appearing in the output corresponds to an unflagged feature in the input MetaboSet object

## Limitations

- The original notame repository is migrated and no longer maintained; active development is at hanhineva-lab/notame, and the package API 'is still quite experimental, and breaking changes are possible'
- MUVR results depend on the repeated double cross-validation settings (nRep, nOuter, nInner, varRatio), so variable-selection sets are not deterministic across parameter choices
- notame 'mainly consists of methods found in other packages', so MUVR behavior and its 'min'/'mid'/'max' semantics are external to notame itself

## Evidence

- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [abstract] notame bundles lab protocols and statistical methods for nutritional metabolomics data: "We provide an overview of lab protocols and statistical methods that we commonly practice for the analysis of nutritional metabolomics data."
- [readme] Data is stored in a custom object that holds all the information about the features and samples along with the feature abundance matrix.: [evidence span withheld — anchor status: external_doc]
