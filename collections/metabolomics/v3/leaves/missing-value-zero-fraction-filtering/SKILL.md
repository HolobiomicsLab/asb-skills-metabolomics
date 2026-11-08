---
name: missing-value-zero-fraction-filtering
description: Use when an LC/MS feature table (e.g. apLCMS or XCMS output on a dataset
  like ST000868) contains missing/zero values and a two-group comparison (e.g. cell-type
  t-test at FDR <= 0.2) requires removing sparse features before testing.
when_to_use_negative:
- The input is already a cleaned feature table with missing values imputed or previously
  filtered — re-filtering would double-remove features
- The analysis goal is batch effect correction of raw preprocessing output rather
  than pre-statistical feature filtering (use the two-stage preprocessing skill instead)
- The feature table comes from a single-group design with no subgroup labels, since
  the <= 33% zero criterion is computed within each biological group
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: apLCMS
  role: Produces the feature tables whose zero-fraction distribution the filter acts
    on; filter is applied to uncorrected apLCMS output before t-testing
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment in which the per-group zero-fraction filter, t-tests, and Benjamini-Hochberg
    FDR correction are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:38a1aa03678935dd926fdb9b300136bee4543cdf80b2b0e42d96b7654734bd87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# missing-value-zero-fraction-filtering

## Summary

A pre-statistical filter for LC/MS metabolomics feature tables that retains only features with <= 33% zero (missing) intensity values within at least one biological group, so that group-comparison tests like the oak vs. wine yeast t-test are performed only on features with usable signal in at least one condition.

## When to use

Use when an LC/MS feature table (e.g. apLCMS or XCMS output on a dataset like ST000868) contains missing/zero values and a two-group comparison (e.g. cell-type t-test at FDR <= 0.2) requires removing sparse features before testing.

## When NOT to use

- The input is already a cleaned feature table with missing values imputed or previously filtered — re-filtering would double-remove features
- The analysis goal is batch effect correction of raw preprocessing output rather than pre-statistical feature filtering (use the two-stage preprocessing skill instead)
- The feature table comes from a single-group design with no subgroup labels, since the <= 33% zero criterion is computed within each biological group

## Inputs

- Uncorrected LC/MS feature data matrix (features x samples) from apLCMS or XCMS preprocessing of the ST000868 dataset
- Sample-to-group label mapping (e.g. oak vs. wine yeast cell-type assignments)

## Outputs

- Filtered feature table containing only features with <= 33% zeros within at least one group
- Downstream counts of significant features (BH FDR <= 0.2) per parameter setting, e.g. fig3d_significant_feature_counts.csv

## How to apply

Load the uncorrected feature data matrix (samples as columns, m/z-RT features as rows) produced by apLCMS or XCMS preprocessing, along with group labels (e.g. oak vs. wine yeast cell types from ST000868). For each feature, compute the fraction of zero values within each group separately, and retain the feature if its zero fraction is <= 33% (i.e. at least ~67% detected samples) in at least one of the two groups. This per-group criterion prevents dropping features that are genuinely present in only one condition, which is often the biologically interesting signal. After filtering, apply the downstream test (two-sample t-test per feature) and Benjamini-Hochberg FDR correction, then count features with adjusted p <= 0.2. The rationale, grounded in the paper's two-stage preprocessing evaluation, is that incomplete weak signal recovery and batch-driven missingness inflate zero counts, so the filter must be applied to the feature table before statistical testing rather than after.

## Related tools

- **apLCMS** (Produces the feature tables whose zero-fraction distribution the filter acts on; filter is applied to uncorrected apLCMS output before t-testing) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment in which the per-group zero-fraction filter, t-tests, and Benjamini-Hochberg FDR correction are implemented)

## Examples

```
keep <- rowSums(ftrs[, oak]==0, na.rm=TRUE) <= 0.33*ncol_oak | rowSums(ftrs[, wine]==0, na.rm=TRUE) <= 0.33*ncol_wine; ftrs.filt <- ftrs[keep,]
```

## Evaluation signals

- Every retained feature has zero fraction <= 0.33 within at least one of the two groups (oak or wine yeast); spot-check a sample of features to confirm
- Features failing the <= 33% zero criterion in both groups are absent from the filtered table
- The counts of significant features (BH FDR <= 0.2) per parameter setting reproduce the qualitative pattern in Fig. 3d (two-stage method tending to detect more significant features than apLCMS or XCMS)
- The output counts table (e.g. fig3d_significant_feature_counts.csv) has one row per apLCMS parameter setting with a non-negative integer significant-feature count
- Re-running the filter on the same input matrix yields an identical retained-feature set (deterministic, no imputation involved)

## Limitations

- The 33% zero-fraction threshold is dataset- and design-specific for this oak vs. wine yeast comparison; it is not a universal cut-off and other studies may need different missingness tolerances
- Applying post hoc batch effect correction cannot recover features lost to misalignment or incomplete weak signal recovery from the original data, so features removed here may reflect preprocessing artifacts rather than true absence
- The filter is defined on the uncorrected feature table; results will differ if applied after batch correction or to a different preprocessing pipeline

## Evidence

- [other] Filter the feature table in R to retain only features with <= 33% zero (missing) values within at least one of the two cell types.: [evidence span withheld — anchor status: not_found]
- [other] For every retained feature, perform a two-sample t-test comparing oak versus wine yeast intensity values.: [evidence span withheld — anchor status: not_found]
- [other] Two-stage method detects more significant features at FDR <= 0.2: [evidence span withheld — anchor status: not_found]
- [intro] Post hoc correction cannot fix weak signal recovery: "They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [discussion] New method improved feature detection consistency: "we showed that the new method improved the consistency of feature detection and quantification"
