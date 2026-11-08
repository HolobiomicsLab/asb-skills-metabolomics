---
name: qc-sample-removal
description: Use when a notame MetaboSet still contains pooled QC samples after flag_detection,
  correct_drift, and flag_quality, and imputation of missing values (e.g. impute_rf)
  is the next preprocessing step.
when_to_use_negative:
- Before drift correction or quality flagging — QCs are required by correct_drift
  and assess_quality, so they must not be removed earlier in the pipeline
- When imputation is not the next step and QC samples are still needed for downstream
  QC metrics or batch correction (e.g. normalize_batches uses QC information)
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides the MetaboSet object model and the drop_qcs function used to remove
    QC samples prior to imputation; imputation itself is done with impute_rf
  repo: https://github.com/hanhineva-lab/notame
- name: missForest
  role: Random-forest imputation backend whose applicability is the reason QCs must
    be dropped first — imputation should be based on biological-sample patterns
- name: R
  role: Environment in which the notame preprocessing workflow (mark_nas, flag_detection,
    correct_drift, drop_qcs, impute_rf) is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:afc2c1bbf10e0fe806cbb84247e4628bd8aa475c9bd2dac8574105d328b5a206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - missForest
  - notame
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# qc-sample-removal

## Summary

Removes QC (pooled quality-control) samples from a notame MetaboSet object with drop_qcs before random-forest imputation, so that missing values are imputed from patterns in the biological samples rather than from QC injections.

## When to use

Use when a notame MetaboSet still contains pooled QC samples after flag_detection, correct_drift, and flag_quality, and imputation of missing values (e.g. impute_rf) is the next preprocessing step.

## When NOT to use

- Before drift correction or quality flagging — QCs are required by correct_drift and assess_quality, so they must not be removed earlier in the pipeline
- When imputation is not the next step and QC samples are still needed for downstream QC metrics or batch correction (e.g. normalize_batches uses QC information)

## Inputs

- Merged MetaboSet object (e.g. from merge_metabosets across LC-MS modes) containing QC samples and a feature-by-sample exprs matrix with NAs
- Sample metadata identifying which rows/samples are QCs

## Outputs

- MetaboSet object restricted to biological samples (QC rows removed), ready for impute_rf
- Saved workspace file such as full_data.RData holding the QC-free (optionally imputed) object

## How to apply

In the notame workflow, QCs are used up front for drift correction (correct_drift) and quality assessment (assess_quality/flag_quality), but after that point they are no longer needed. Before imputing missing values, call drop_qcs on the merged MetaboSet to subset to biological samples only; imputation such as impute_rf (missForest-based random-forest imputation) must be based on patterns in the biological data, so leaving QCs in would distort the learned feature profiles. Retain the pre-drop object dimensions so the result can be validated. Save the resulting imputed, QC-free object (e.g. full_data.RData) for downstream statistical analysis.

## Related tools

- **notame** (Provides the MetaboSet object model and the drop_qcs function used to remove QC samples prior to imputation; imputation itself is done with impute_rf) — https://github.com/hanhineva-lab/notame
- **missForest** (Random-forest imputation backend whose applicability is the reason QCs must be dropped first — imputation should be based on biological-sample patterns)
- **R** (Environment in which the notame preprocessing workflow (mark_nas, flag_detection, correct_drift, drop_qcs, impute_rf) is executed)

## Examples

```
imputed <- impute_rf(drop_qcs(merged_data), seed = 1)  # drop QCs before missForest-based imputation; save as full_data.RData
```

## Evaluation signals

- The QC-free MetaboSet's sample dimension equals the pre-drop object's dimension minus the number of QC samples; feature dimension is unchanged
- After subsequent impute_rf, the exprs matrix contains no remaining NA values among non-flagged (good-quality) features
- Sample and feature dimensions of the imputed object match the pre-imputation, QC-dropped object
- The saved artifact (e.g. full_data.RData) exists and loads as a MetaboSet

## Limitations

- QCs must be removed only after quality assessment and drift correction; dropping them earlier breaks correct_drift and assess_quality/flag_quality
- The notame package API is experimental and breaking changes are possible; the antonvsdata/notame repository is migrated and no longer maintained (active development is at hanhineva-lab/notame)
- impute_rf with all_features = FALSE operates only on good-quality unflagged features, so flagged features may retain NAs

## Evidence

- [full_text] Missing values are imputed by random forest imputation (missForest), and QC samples must first be removed so that the imputation is based on patterns in the biological data.: [evidence span withheld — anchor status: not_found]
- [methods] impute_rf for random forest imputation: "- ```impute_rf``` for random forest imputation"
- [readme] Imputing missing values, multiple strategies available. Random forest imputation recommended: [evidence span withheld — anchor status: external_doc]
- [methods] To run drift correction on your data, use the function ```correct_drift```.: "To run drift correction on your data, use the function ```correct_drift```."
- [readme] The package API is still quite experimental, and breaking changes are possible: [evidence span withheld — anchor status: external_doc]
