---
name: repeated-double-cross-validation-variable-selection
description: Use when a preprocessed, imputed LC-MS metabolomics feature table (MetaboSet
  object) with a known class label (e.g. a 'group' column) is available and the goal
  is supervised ranking of molecular features into minimal-optimal vs. all-relevant
  subsets.
when_to_use_negative:
- The data are raw LC-MS files or an unprocessed peak table — notame's preprocessing
  (mark_nas, correct_drift, flagging, imputation) must run first
- No class label exists and the goal is unsupervised exploration rather than supervised
  variable selection
- You need only feature-wise univariate tests rather than multivariate prediction/variable-selection
  models
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: notame
  role: Provides the preprocessed MetaboSet object and the workflow context (flagged
    features, imputation) that muvr_analysis consumes
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: R package providing repeated double cross-validation RF and PLS(-DA) models
    and the min/mid/max variable-selection sets
- name: R
  role: Runtime environment in which notame and MUVR are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:1f92f93492bfd530b87ef893cb471d5991499025c2fa7614fc848fdfe8732842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Repeated double cross-validation variable selection

## Summary

Applies MUVR's repeated double cross-validation with random forest (RF) and PLS(-DA) models to a preprocessed, imputed MetaboSet object from the notame LC-MS workflow, yielding nested 'min', 'mid' and 'max' variable-selection sets that distinguish minimal-optimal from all-relevant molecular features.

## When to use

Use when a preprocessed, imputed LC-MS metabolomics feature table (MetaboSet object) with a known class label (e.g. a 'group' column) is available and the goal is supervised ranking of molecular features into minimal-optimal vs. all-relevant subsets.

## When NOT to use

- The data are raw LC-MS files or an unprocessed peak table — notame's preprocessing (mark_nas, correct_drift, flagging, imputation) must run first
- No class label exists and the goal is unsupervised exploration rather than supervised variable selection
- You need only feature-wise univariate tests rather than multivariate prediction/variable-selection models

## Inputs

- Preprocessed, imputed MetaboSet object (e.g. full_data.RData) from the notame project-example pipeline, with quality-flagged features marked
- Class label column (e.g. the 'group' column) defining the supervised prediction target
- MUVR cross-validation settings: nRep, nOuter, nInner, varRatio

## Outputs

- 'min', 'mid' and 'max' variable-selection feature sets per model (RF and PLS(-DA))
- ranked_features_muvr.csv: tidy table with model, variable-selection set, Feature_ID and importance-rank columns

## How to apply

Load the imputed MetaboSet object (e.g. full_data.RData from the notame preprocessing pipeline) with notame in R, and restrict analysis to unflagged features — notame's flag_detection, flag_contaminants and flag_quality steps have already removed low-quality features. Run muvr_analysis on the unflagged features with the class label y (the group column), supplying MUVR's repeated double cross-validation parameters (nRep, nOuter, nInner, varRatio) and method so both RF and PLS(-DA) models are fitted with outer and inner CV loops. Extract the three variable-selection model sets MUVR returns per model: 'min' (minimal-optimal), 'mid' and 'max' (all-relevant), representing increasingly inclusive feature subsets. Rank features within each model/set combination by model-specific importance — VIP scores for PLS(-DA) and permutation error-rate increase for RF (analogous to notame's importance_rf helper) — and write model, set, Feature_ID and importance rank per row to a tidy CSV such as ranked_features_muvr.csv.

## Related tools

- **notame** (Provides the preprocessed MetaboSet object and the workflow context (flagged features, imputation) that muvr_analysis consumes) — https://github.com/hanhineva-lab/notame
- **MUVR** (R package providing repeated double cross-validation RF and PLS(-DA) models and the min/mid/max variable-selection sets)
- **R** (Runtime environment in which notame and MUVR are executed)

## Examples

```
muvr_analysis(full_data, y = "group", method = c("RF", "PLS"), nRep = nRep, nOuter = nOuter, nInner = nInner, varRatio = varRatio)
```

## Evaluation signals

- ranked_features_muvr.csv exists and parses as a table with model / set / Feature_ID / importance-rank columns
- The output contains ranked entries for both RF and PLS(-DA) across all three ('min', 'mid', 'max') selection sets
- Only unflagged features appear in the ranked table (features removed by flag_detection / flag_contaminants / flag_quality are excluded)
- Importance ranks are internally consistent: PLS ranks reflect VIP scores and RF ranks reflect permutation error-rate increase
- The 'max' (all-relevant) set is at least as large as the 'min' (minimal-optimal) set for each model

## Limitations

- notame's API was described as 'still quite experimental' with breaking changes possible; the original antonvsdata/notame repository is no longer maintained (migrated to hanhineva-lab/notame)
- The skill presumes the notame preprocessing pipeline (drift correction, flagging, imputation) has already been applied; applying it to unimputed or unflagged data undermines model quality
- Model stability depends on the chosen repeated double cross-validation parameters (nRep, nOuter, nInner, varRatio)

## Evidence

- [abstract] abstract-intro: "we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches"
- [readme] readme-multivariate: [evidence span withheld — anchor status: external_doc]
- [intro] intro-preprocessed: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
