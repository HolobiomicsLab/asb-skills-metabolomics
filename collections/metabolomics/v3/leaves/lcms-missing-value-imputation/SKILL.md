---
name: lcms-missing-value-imputation
description: Use when a non-targeted LC-MS peak table stored as a notame MetaboSet
  still contains NA values in its exprs matrix after drift correction, quality flagging,
  and QC removal, and those gaps must be filled before statistical analysis.
when_to_use_negative:
- When QC samples are still present in the object — remove them with drop_qcs first,
  since imputation should be based on biological samples only.
- When the data is not preprocessed LC-MS peak data (e.g. raw instrument files) —
  notame's imputation operates on MetaboSet abundance matrices, not raw LC-MS files.
- When you intend to leave missing values as NA for methods that tolerate them — notame
  treats NA as the missing-value representation and imputation is a preprocessing
  step before its statistical tests.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: notame
  role: Supplies mark_nas, drop_qcs, impute_rf and impute_simple functions operating
    on MetaboSet objects for LC-MS data preprocessing including missing-value imputation.
  repo: https://github.com/hanhineva-lab/notame
- name: missForest
  role: Backend random forest imputation engine used by notame's impute_rf; recommended
    per Kokla et al. comparison of imputation strategies.
- name: R
  role: Runtime environment; notame is an R package and NA is R's native missing-value
    representation used throughout imputation.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:977cd97f3cf0016c7be38427e4b9d69c02fb8eda7aba4947dff88a2f3ef646e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - missForest
  - notame
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lcms-missing-value-imputation

## Summary

Imputes remaining missing values (NAs) in a notame MetaboSet feature-abundance matrix using random forest imputation via missForest, after QC samples have been dropped and low-quality features flagged.

## When to use

Use when a non-targeted LC-MS peak table stored as a notame MetaboSet still contains NA values in its exprs matrix after drift correction, quality flagging, and QC removal, and those gaps must be filled before statistical analysis.

## When NOT to use

- When QC samples are still present in the object — remove them with drop_qcs first, since imputation should be based on biological samples only.
- When the data is not preprocessed LC-MS peak data (e.g. raw instrument files) — notame's imputation operates on MetaboSet abundance matrices, not raw LC-MS files.
- When you intend to leave missing values as NA for methods that tolerate them — notame treats NA as the missing-value representation and imputation is a preprocessing step before its statistical tests.

## Inputs

- MetaboSet object (post drift correction and quality flagging) whose exprs matrix contains NA values
- Sample metadata distinguishing biological samples from pooled QC samples
- Output of mark_nas (zero abundances converted to NA)

## Outputs

- MetaboSet object with imputed exprs matrix (no NAs among imputable features)
- Saved R data file, e.g. full_data.RData, ready for statistical analysis

## How to apply

First mark zero abundances as NA with mark_nas so missingness is represented by NA, as required before notame's imputation functions. Complete drift correction (correct_drift), flag low-quality features (flag_detection, flag_quality), and drop the QC samples with drop_qcs, because the article states QC samples are removed before imputation so the random forest learns only from biological data patterns. Then call impute_rf, which wraps the missForest package for random forest imputation recommended by Kokla et al.; with the default all_features = FALSE it imputes only unflagged, good-quality features. Keep a reproducible seed and save the resulting MetaboSet (e.g. full_data.RData) for downstream feature-wise tests and multivariate models. If random forest is too slow, notame also offers impute_simple for simpler imputation strategies.

## Related tools

- **notame** (Supplies mark_nas, drop_qcs, impute_rf and impute_simple functions operating on MetaboSet objects for LC-MS data preprocessing including missing-value imputation.) — https://github.com/hanhineva-lab/notame
- **missForest** (Backend random forest imputation engine used by notame's impute_rf; recommended per Kokla et al. comparison of imputation strategies.)
- **R** (Runtime environment; notame is an R package and NA is R's native missing-value representation used throughout imputation.)

## Examples

```
set.seed(42); imputed <- impute_rf(full_data, all_features = FALSE); saveRDS(imputed, "full_data.RData")
```

## Evaluation signals

- After imputation, the exprs matrix of the resulting MetaboSet contains no NA values among unflagged (imputable) features.
- Sample and feature dimensions of the imputed object match the pre-imputation, QC-dropped MetaboSet.
- QC samples were removed (drop_qcs) before impute_rf was invoked — verify by checking that pooled QC sample IDs are absent from sample metadata.
- The output file (e.g. full_data.RData) loads back as a valid MetaboSet for downstream statistical analysis.
- A reproducible random seed was set so re-running impute_rf yields identical imputed values.

## Limitations

- Random forest imputation via missForest can be computationally intensive on large feature matrices; notame offers impute_simple with simpler strategies as an alternative.
- With the default all_features = FALSE, only good-quality unflagged features are imputed — flagged low-quality features remain as-is and may still contain NAs.
- The notame package API is described as 'quite experimental' with possible breaking changes; the original antonvsdata/notame repository is migrated and no longer updated (active development at hanhineva-lab/notame).
- Imputation must not be run before quality flagging and QC removal, or the random forest will learn from QC-sample and poor-quality feature patterns.

## Evidence

- [methods] Missing values are imputed by random forest imputation (missForest), and QC samples must first be removed so that the imputation is based on patterns in the biological data.: [evidence span withheld — anchor status: not_found]
- [methods] notame's imputation functions are documented as operating on the exprs matrix of MetaboSet objects with NA representation.: "In R, missing values are represented by a special value ```NA```"
- [methods] notame's imputation functions are listed alongside simple alternatives.: "- ```impute_rf``` for random forest imputation"
- [methods] notame's simple imputation alternatives are documented in the methods section.: "If you don't want to use random forest imputation, there are other simpler alternatives implemented in this package"
- [readme] Imputation is listed as a core notame functionality after flagging low-quality features.: [evidence span withheld — anchor status: external_doc]
