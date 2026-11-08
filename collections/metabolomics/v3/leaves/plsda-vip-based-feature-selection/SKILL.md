---
name: plsda-vip-based-feature-selection
description: Use when a preprocessed, imputed MetaboSet object from the notame workflow
  is available and the goal is supervised ranking of molecular features (e.g. by group
  label) using PLS-DA VIP scores or RF permutation importance.
when_to_use_negative:
- Input is a raw LC-MS file (e.g. mzML/MS-DIAL peak picking not yet done) — the skill
  requires a preprocessed MetaboSet object
- Missing values have not been imputed (e.g. by random forest via notame's impute_rf)
  and low-quality features have not been flagged
- The analysis goal is unsupervised exploration (clustering similar features) rather
  than supervised feature ranking against a class label
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: notame
  role: Provides the MetaboSet object, preprocessing (mark_nas, correct_drift, flag_quality,
    impute_rf) and the wrapper for running multivariate analysis on unflagged features
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: Fits PLS-DA and random forest models via repeated double cross-validation
    and returns 'min', 'mid' and 'max' variable-selection sets used for ranking features
- name: R
  role: Execution environment for the notame package and MUVR analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:978f5e14080a6795d7c6709409018ef1d953ecf2c1f09be8927c96c37ac49dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# PLS-DA VIP-based feature selection

## Summary

Selects the most interesting molecular features in non-targeted LC-MS metabolomics by fitting PLS-DA (and optionally random forest) models via MUVR and ranking features by model-specific importance, yielding 'min', 'mid' and 'max' variable-selection sets.

## When to use

Use when a preprocessed, imputed MetaboSet object from the notame workflow is available and the goal is supervised ranking of molecular features (e.g. by group label) using PLS-DA VIP scores or RF permutation importance.

## When NOT to use

- Input is a raw LC-MS file (e.g. mzML/MS-DIAL peak picking not yet done) — the skill requires a preprocessed MetaboSet object
- Missing values have not been imputed (e.g. by random forest via notame's impute_rf) and low-quality features have not been flagged
- The analysis goal is unsupervised exploration (clustering similar features) rather than supervised feature ranking against a class label

## Inputs

- Preprocessed, imputed MetaboSet object (e.g. full_data.RData) from the notame LC-MS pipeline
- Sample metadata column containing the class label (e.g. group) used as the response
- Flag information identifying low-quality molecular features to exclude

## Outputs

- Ranked feature table (e.g. ranked_features_muvr.csv) with columns model, variable-selection set, Feature_ID and importance rank
- Per-model variable-selection sets: 'min' (minimal-optimal), 'mid' and 'max' (all-relevant) feature subsets for both RF and PLS-DA

## How to apply

Load the preprocessed, imputed MetaboSet object (e.g. full_data.RData produced by the notame preprocessing pipeline) with the notame package in R, after low-quality features have been flagged. Run MUVR's multivariate analysis on the unflagged features using the class label (e.g. the group column) as the response, with repeated double cross-validation parameters (nRep, nOuter, nInner, varRatio) so that both random forest (RF) and PLS-DA models are fitted. Extract the three variable-selection model sets per analysis — the minimal-optimal ('min'), 'mid' and all-relevant ('max') models. Rank features within each model and selection set by model-specific importance: VIP scores for PLS-DA and permutation error-rate increase for RF. Assemble a tidy table with model, selection set, Feature_ID and importance rank per row and write it out (e.g. ranked_features_muvr.csv) for downstream metabolite identification.

## Related tools

- **notame** (Provides the MetaboSet object, preprocessing (mark_nas, correct_drift, flag_quality, impute_rf) and the wrapper for running multivariate analysis on unflagged features) — https://github.com/hanhineva-lab/notame
- **MUVR** (Fits PLS-DA and random forest models via repeated double cross-validation and returns 'min', 'mid' and 'max' variable-selection sets used for ranking features)
- **R** (Execution environment for the notame package and MUVR analyses)

## Examples

```
Rscript -e 'library(notame); load("full_data.RData"); res <- muvr_analysis(full_data, y = "group", method = "P", nRep = 5, nOuter = 6, nInner = 5); write.csv(res, "ranked_features_muvr.csv", row.names = FALSE)'
```

## Evaluation signals

- The output CSV parses as a table with model/set/Feature_ID/importance-rank columns
- Ranked entries exist for both RF and PLS-DA across all three ('min', 'mid', 'max') selection sets
- No low-quality (flagged) features appear in the ranked table — only unflagged features were modeled
- Each Feature_ID in the ranked table corresponds to a feature present in the input MetaboSet object's feature metadata
- PLS-DA rankings are VIP-score based and RF rankings are permutation error-rate based, as recorded in the model column

## Limitations

- MUVR results depend on repeated double cross-validation settings (nRep, nOuter, nInner, varRatio); unstable rankings can result from too few repetitions
- The notame package API is still quite experimental and breaking changes are possible
- Only features that pass quality flagging (detection, contaminants, other quality metrics) are suitable for supervised modeling

## Evidence

- [methods] MUVR yields min/mid/max variable-selection model sets with VIP scores for PLS-DA and permutation error-rate increase for random forest: [evidence span withheld — anchor status: not_found]
- [intro] notame is used on preprocessed LC-MS data: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [methods] notame flags low-quality molecular features: "LC-MS metabolomics datasets contain a significant number of low-quality features"
- [readme] notame README: imputation of missing values via random forest is recommended: [evidence span withheld — anchor status: external_doc]
- [readme] notame README: visualizations and statistical analyses for QC and interpretation of results: [evidence span withheld — anchor status: external_doc]
