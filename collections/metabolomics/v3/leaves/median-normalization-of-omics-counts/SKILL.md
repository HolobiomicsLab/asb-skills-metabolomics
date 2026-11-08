---
name: median-normalization-of-omics-counts
description: Decision support for choosing and applying a sample-scaling normalization
  (median vs EigenMS/NOMIS/CRMN) on MultiABLER-processed LC-MS/MS feature tables from
  metabolomic or lipidomic experiments.
when_to_use_negative:
- Input is already a normalized feature table or the user requires kNN imputation,
  mean normalization, or quantile normalization, none of which are implemented in
  MultiABLER
- Analysis is proteomic; MultiABLER does not provide functions to analyze proteomic
  data
- Small feature sets where EigenMS-style batch correction is unnecessary and simple
  median scaling is sufficient without benchmarking
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MultiABLER
  role: R pipeline that provides normMedian for median normalization, plus tbManipulate
    for upstream filtering/imputation/log-transformation of the feature table
  repo: https://github.com/holab-hku/MultiABLER
- name: ClassifyR
  role: Cross-validation framework used to benchmark the downstream classification
    performance of each normalization variant
- name: EigenMS
  role: Surrogate-variable-based normalization benchmarked against median normalization;
    best-performing variant on the liver lipidomic data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:b07d05e1525ac3a9f7f7b524bce9093a4fe927fd53abe6b52817a80c063c82bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - ClassifyR
  - EigenMS
  - MultiABLER
  - R
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# median-normalization-of-omics-counts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Median normalization of metabolomic/lipidomic feature-intensity tables, implemented in MultiABLER (R), which scales each sample by its median feature intensity to reduce sample-level technical variation before downstream differential expression or classification analysis.

## When to use

Decision support for choosing and applying a sample-scaling normalization (median vs EigenMS/NOMIS/CRMN) on MultiABLER-processed LC-MS/MS feature tables from metabolomic or lipidomic experiments.

## When NOT to use

- Input is already a normalized feature table or the user requires kNN imputation, mean normalization, or quantile normalization, none of which are implemented in MultiABLER
- Analysis is proteomic; MultiABLER does not provide functions to analyze proteomic data
- Small feature sets where EigenMS-style batch correction is unnecessary and simple median scaling is sufficient without benchmarking

## Inputs

- Peak-intensity feature table (rows = metabolite/lipid features, columns = samples) from MultiABLER extraction
- Sample metadata / class labels for downstream classification or differential analysis

## Outputs

- Normalized feature-intensity table (same dimensions, per-sample median scaling applied)
- Benchmark metrics (median balanced accuracy, median AUC) when used in the normalization-comparison workflow

## How to apply

First process raw feature intensities with MultiABLER's tbManipulate() to apply the paper's feature-filtering threshold, imputation, and log-transformation settings. Then choose a normalization variant: no normalization, median normalization via normMedian, EigenMS, NOMIS, CRMN, or EigenMS followed by median normalization. In the paper's benchmark on the 35-sample liver lipidomic dataset (positive mode, S-BSST1038), each variant was fed into ClassifyR cross-validation with DLDA (5 folds, 20 repetitions); EigenMS was best (median BA 0.68, AUC 0.75), with EigenMS followed by median normalization essentially tied (BA 0.69, AUC 0.76), while median normalization alone and no normalization performed worse. Median normalization is therefore a reasonable baseline and the simple second stage after EigenMS, but on MultiABLER-style data it should be benchmarked against EigenMS rather than assumed best. The output feature table is then suitable for differential expression, sPLS, or rCCA analyses.

## Related tools

- **MultiABLER** (R pipeline that provides normMedian for median normalization, plus tbManipulate for upstream filtering/imputation/log-transformation of the feature table) — https://github.com/holab-hku/MultiABLER
- **ClassifyR** (Cross-validation framework used to benchmark the downstream classification performance of each normalization variant)
- **EigenMS** (Surrogate-variable-based normalization benchmarked against median normalization; best-performing variant on the liver lipidomic data)

## Examples

```
df_norm <- normMedian(tbManipulate(df, filter_threshold, impute, log_base))
```

## Evaluation signals

- After normMedian, the median feature intensity of each sample column should be equal across samples (per-sample medians collapse to a common value).
- Median balanced accuracy and AUC from ClassifyR (DLDA, 5 folds, 20 reps) should be computed for each variant and compared against the published benchmarks (BA: 0.40 none, 0.68 EigenMS, 0.48 NOMIS, 0.38 CRMN, 0.69 EigenMS+median; AUC: 0.35, 0.75, 0.48, 0.37, 0.76) within ~±0.05.
- EigenMS-based variants should rank among the top performers; if median normalization alone outperforms EigenMS substantially, re-check tbManipulate filtering/imputation settings.
- Downstream suitability check: the normalized feature table should still support differential expression, sPLS, and rCCA analyses without further transformation.

## Limitations

- kNN imputation, mean normalization, and quantile normalization are not implemented in MultiABLER.
- MultiABLER does not provide functions to analyze proteomic data; proteomic integration is planned for future development.
- Median normalization alone underperformed EigenMS on the liver lipidomic benchmark; it should not be assumed to be the best method for MultiABLER-processed data.

## Evidence
