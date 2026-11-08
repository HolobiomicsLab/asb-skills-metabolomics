---
name: random-forest-matrix-completion
description: Use when a preprocessed LC-MS MetaboSet object still contains NA values
  in exprs and a statistically sound fill-in (random forest, per Kokla et al.) is
  needed before feature-wise tests or multivariate models.
when_to_use_negative:
- When the missingness pattern should drive feature filtering instead of imputation
  — notame first flags low-quality features with flag_detection/flag_quality and only
  imputes what remains; do not impute instead of flagging.
- When QC samples must remain in the object (e.g. ongoing batch correction such as
  normalize_batches/ruvs_qc, which rely on QC signal) — impute_rf is applied only
  after QCs are dropped.
- When a deterministic simple strategy is preferred or random forest is too slow for
  the feature count — impute_simple offers simpler alternatives.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides impute_rf (random-forest imputation wrapper) and impute_simple, plus
    drop_qcs and flag_quality used to prepare the matrix before imputation.
  repo: https://github.com/hanhineva-lab/notame
- name: missForest
  role: The underlying random-forest imputation engine invoked by impute_rf to iteratively
    complete the feature-abundance matrix.
- name: R
  role: Execution environment; missing values are represented as NA in the exprs matrix
    that impute_rf completes.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:6e3a4ebf10bf2d8f699f3630633a90151e895ae6cc77bc90d44ebeb1928b281d
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# random-forest-matrix-completion

## Summary

Fills missing values in a MetaboSet feature-abundance matrix using random-forest imputation (missForest), following the notame workflow, after quality-flagged features and QC samples have been set aside.

## When to use

Use when a preprocessed LC-MS MetaboSet object still contains NA values in exprs and a statistically sound fill-in (random forest, per Kokla et al.) is needed before feature-wise tests or multivariate models.

## When NOT to use

- When the missingness pattern should drive feature filtering instead of imputation — notame first flags low-quality features with flag_detection/flag_quality and only imputes what remains; do not impute instead of flagging.
- When QC samples must remain in the object (e.g. ongoing batch correction such as normalize_batches/ruvs_qc, which rely on QC signal) — impute_rf is applied only after QCs are dropped.
- When a deterministic simple strategy is preferred or random forest is too slow for the feature count — impute_simple offers simpler alternatives.

## Inputs

- Merged MetaboSet object (post drift correction and quality flagging) with NA values in the exprs abundance matrix
- Quality flags already assigned by flag_detection/flag_quality (so all_features can remain FALSE)
- QC samples that can be removed via drop_qcs prior to imputation

## Outputs

- MetaboSet object whose exprs matrix has no remaining NA values among non-flagged features
- Saved imputed dataset, e.g. full_data.RData, for downstream statistical analysis

## How to apply

After detection flags, drift correction (correct_drift), and quality flagging (flag_quality) are done, first drop QC samples with drop_qcs so imputation learns from biological samples only, since QCs are no longer needed once quality assessment is finished. Then call impute_rf on the merged MetaboSet: by default all_features = FALSE so only good-quality, unflagged features are imputed; the missForest algorithm iteratively fills NA entries (those marked via mark_nas, e.g. zeros set to NA) using all other measured features as predictors. Set a reproducible seed beforehand because missForest is stochastic. Simple alternatives exist (impute_simple) but notame recommends random forest based on Kokla et al.'s benchmarking of imputation strategies for metabolomics data. Persist the imputed object (e.g. full_data.RData) as the input to downstream statistical analysis.

## Related tools

- **notame** (Provides impute_rf (random-forest imputation wrapper) and impute_simple, plus drop_qcs and flag_quality used to prepare the matrix before imputation.) — https://github.com/hanhineva-lab/notame
- **missForest** (The underlying random-forest imputation engine invoked by impute_rf to iteratively complete the feature-abundance matrix.)
- **R** (Execution environment; missing values are represented as NA in the exprs matrix that impute_rf completes.)

## Examples

```
set.seed(12345); imputed <- impute_rf(drop_qcs(full_data), all_features = FALSE); save(imputed, file = 'full_data.RData')
```

## Evaluation signals

- After imputation, the exprs matrix of the returned MetaboSet contains no NA entries for features that were not flagged as low quality.
- Sample and feature dimensions of the imputed object match the pre-imputation, QC-dropped MetaboSet exactly.
- Imputation was run after drop_qcs — verify QCs are absent from the imputed object's sample metadata.
- all_features = FALSE behavior: flagged (low-quality) features can still contain NA after impute_rf, confirming imputation was restricted to good-quality features.
- The imputed object is saved (e.g. full_data.RData) and downstream statistical functions receive a NA-free matrix for unflagged features.

## Limitations

- Random forest imputation (missForest) is stochastic; a reproducible seed must be set or results will vary between runs.
- The notame package API is described as 'still quite experimental, and breaking changes are possible'.
- The antonvsdata/notame repo is archived — development moved to hanhineva-lab/notame; issues opened there are not monitored.
- Imputation is recommended only for good-quality features; imputing poor-quality flagged features (all_features = TRUE) is not the recommended path.

## Evidence

- [methods] Imputation via missForest: "If you don't want to use random forest imputation, there are other simpler alternatives implemented in this package"
- [methods] impute_rf is the random forest wrapper: "- ```impute_rf``` for random forest imputation"
- [methods] missing values as NA: "In R, missing values are represented by a special value ```NA```"
- [methods] mark_nas creates the NAs impute_rf fills: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
- [readme] Random forest recommended (README): [evidence span withheld — anchor status: external_doc]
- [readme] notame bundles preprocessing incl. imputation: "We use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data"
