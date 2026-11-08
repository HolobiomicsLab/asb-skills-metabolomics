---
name: scaling-normalization-of-metabolite-abundance-matrices
description: Use when a log-transformed featuredata matrix (samples as rows, metabolites
  as columns) shows sample-wide intensity differences—e.g. dilution or loading effects—and
  no QC samples or internal standards are needed for correction.
when_to_use_negative:
- The correction should exploit QC samples analysed periodically throughout the batch
  (use NormQcsamples/QC-RLSC instead).
- Internal standards or quality-control metabolites are flagged in metabolitedata
  and should drive the normalization (use NormQcmets methods 'nomis', 'ccmn', 'is',
  'ruv2').
- A combination of methods, e.g. 'rlsc' followed by 'median', is desired (use NormCombined).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NormalizeMets
  role: Provides the NormScaling function implementing the 'median', 'mean', 'sum',
    and 'ref' scaling methods
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which NormalizeMets is executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:6a0f55595f314768671fd7a9853a5357e1cfb78d0226baa8a393db5e4b093a31
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - SMART 1.0
  - SMART 2.0
  - statTarget
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolite-abundance-normalization
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Scaling normalization of metabolite abundance matrices

## Summary

Sample-wise scaling normalization of a log-transformed metabolomics featuredata matrix using NormalizeMets' NormScaling, which divides each sample by its median, mean, total sum, or a reference vector to remove unwanted sample-level variation.

## When to use

Use when a log-transformed featuredata matrix (samples as rows, metabolites as columns) shows sample-wide intensity differences—e.g. dilution or loading effects—and no QC samples or internal standards are needed for correction.

## When NOT to use

- The correction should exploit QC samples analysed periodically throughout the batch (use NormQcsamples/QC-RLSC instead).
- Internal standards or quality-control metabolites are flagged in metabolitedata and should drive the normalization (use NormQcmets methods 'nomis', 'ccmn', 'is', 'ruv2').
- A combination of methods, e.g. 'rlsc' followed by 'median', is desired (use NormCombined).

## Inputs

- Log-transformed, imputed featuredata matrix (samples as rows, metabolite peak intensities as columns)
- Optional refvec reference vector for method='ref'

## Outputs

- Normalized featuredata matrix (one per scaling method)
- CSV-exported normalized data matrix when saveoutput=TRUE

## How to apply

Prepare the matrix in featuredata format (unique sample names as row names, metabolite names as column names), log transformed via LogTransform (zerotona=TRUE) and imputed with MissingValues (method='knn', k=10, feature.cutoff=0.8, sample.cutoff=0.8) so scaling is not distorted by zeros or missing values. Call NormScaling with one of the four supported methods: 'median' or 'mean' (divide each sample by its median/mean peak intensity), 'sum' (normalization to a total sum), or 'ref' (scaling to a supplied reference vector refvec). Choose 'ref' when a stable reference metabolite or spike-in is available; otherwise median/mean scaling is robust to composition effects. Set saveoutput=TRUE with an outputname to write the normalized matrix, then feed the result into evaluation functions such as LinearModelFit residual RLA plots to judge whether unwanted sample-level variation was removed.

## Related tools

- **NormalizeMets** (Provides the NormScaling function implementing the 'median', 'mean', 'sum', and 'ref' scaling methods) — https://github.com/metabolomicstats/NormalizeMets
- **R** (Runtime environment (version 3.4.3 or higher) in which NormalizeMets is executed) — https://cran.r-project.org/

## Examples

```
NormScaling(featuredata = imp$featuredata, method = "median", saveoutput = TRUE, outputname = "median_norm")
```

## Evaluation signals

- RLA (relative log abundance) plots of the normalized matrix show reduced between-sample spread compared to the pre-scaling log data.
- LinearModelFit p-value distributions on the normalized data behave as expected for the study design (no inflation from residual sample effects).
- Every sample receives a finite, non-zero scale factor; no NaNs are introduced on the imputed matrix.
- Normalized output matrix retains the original dimensions (same samples and metabolites) and is reproducibly written when saveoutput=TRUE.

## Limitations

- Scaling methods correct only sample-level intensity differences; they do not correct within-batch drift (use NormQcsamples/QC-RLSC for that) or leverage QC metabolites.
- Missing values must be handled first (MissingValues with knn, k=10, cutoffs 0.8); scaling on raw data with zeros/missings gives misleading scale factors.
- Method choice ('median' vs 'mean' vs 'sum' vs 'ref') is dataset- and question-dependent and should be justified by comparative evaluation, not assumed.

## Evidence

- [methods] Scaling method description: [evidence span withheld — anchor status: not_found]
- [readme] Input format requirement: [evidence span withheld — anchor status: external_doc]
