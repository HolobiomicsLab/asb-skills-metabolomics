---
name: internal-standard-qc-metabolite-normalization
description: Use when the metabolitedata frame flags internal standards or QC metabolites
  (e.g. IS==1) and the goal is metabolite-level normalization of featuredata via a
  single standard (is), nomis, ccmn, or RUV-based methods (ruv2, ruvrand, ruvrandclust).
when_to_use_negative:
- The dataset lacks metabolitedata annotations designating internal/external standards
  or QC metabolites (IS flags) — use NormScaling (median/mean/sum/ref) or NormQcsamples
  QC-RLSC signal correction instead.
- The aim is correcting drift from QC samples analysed periodically throughout the
  batch (QC-RLSC via NormQcsamples) rather than metabolite-based normalization.
- The input is raw unannotated peak intensities that have not yet been log-transformed
  and missing-value imputed (LogTransform zerotona=TRUE, knn k=10 with 0.8 cutoffs).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NormalizeMets
  role: Provides the NormQcmets function that implements the is, nomis, ccmn, ruv2,
    ruvrand and ruvrandclust metabolite-based normalization methods
  repo: metabolomicstats/NormalizeMets
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which NormalizeMets is installed
    and NormQcmets is executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:b45d2a3d3ed3bb317947ffc79f3b225acd364f904accbb55c8e54a39125c8673
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# internal-standard-qc-metabolite-normalization

## Summary

Apply the NormalizeMets NormQcmets family of normalization methods, which use internal, external standards and other quality control metabolites to remove unwanted variation from a metabolomics feature matrix. Covers the six dispatched methods: is, nomis, ccmn, ruv2, ruvrand, and ruvrandclust.

## When to use

Use when the metabolitedata frame flags internal standards or QC metabolites (e.g. IS==1) and the goal is metabolite-level normalization of featuredata via a single standard (is), nomis, ccmn, or RUV-based methods (ruv2, ruvrand, ruvrandclust).

## When NOT to use

- The dataset lacks metabolitedata annotations designating internal/external standards or QC metabolites (IS flags) — use NormScaling (median/mean/sum/ref) or NormQcsamples QC-RLSC signal correction instead.
- The aim is correcting drift from QC samples analysed periodically throughout the batch (QC-RLSC via NormQcsamples) rather than metabolite-based normalization.
- The input is raw unannotated peak intensities that have not yet been log-transformed and missing-value imputed (LogTransform zerotona=TRUE, knn k=10 with 0.8 cutoffs).

## Inputs

- featuredata: samples-by-metabolites matrix of metabolite peak intensities (unique sample row names, unique metabolite column names)
- metabolitedata: dataframe with metabolite-specific annotations including designation as internal/external standards (IS flags)
- sampledata: dataframe with sample-specific factors (e.g. gender, Age, bmi, run order)
- isvec: numeric vector of a single internal standard peak (for method='is')
- factormat: model matrix of factors of interest, e.g. model.matrix(~gender+Age+bmi, sampledata) (for ruv2)

## Outputs

- normalized featuredata matrix with the same samples-by-metabolites dimensions as the input
- method-specific extras: factors/ncomp, lambda, and estimated unwanted-variation factors for ruv2/ruvrand/ruvrandclust

## How to apply

Preprocess the featuredata matrix by log-transforming with LogTransform (zerotona=TRUE) and imputing remaining missing values with MissingValues using the knn method (k=10, feature.cutoff=0.8, sample.cutoff=0.8). NormQcmets parses its method argument and dispatches against exactly six options: c('is','nomis','ccmn','ruv2','ruvrand','ruvrandclust'); route 'is' to a single internal standard supplied as isvec (e.g. the first internal standard), route 'nomis' and 'ccmn' using qcmets=which(metabolitedata$IS==1), with ccmn additionally given factors=sampledata$gender, and route 'ruv2' using factormat=model.matrix(~gender+Age+bmi, sampledata) with k=2. Each route returns a normalized featuredata matrix plus method-specific extras (factors/ncomp, lambda, and estimated unwanted-variation factors W for the ruv methods). Validate that the dispatch accepts only the six documented strings and that the returned matrix preserves the samples-by-metabolites dimensions of the input.

## Related tools

- **NormalizeMets** (Provides the NormQcmets function that implements the is, nomis, ccmn, ruv2, ruvrand and ruvrandclust metabolite-based normalization methods) — metabolomicstats/NormalizeMets
- **R** (Runtime environment (version 3.4.3 or higher) in which NormalizeMets is installed and NormQcmets is executed) — https://cran.r-project.org/

## Examples

```
norm <- NormQcmets(featuredata=imp$featuredata, factors=sampledata$gender, method="ccmn", qcmets=which(metabolitedata$IS==1))
```

## Evaluation signals

- The method dispatch accepts exactly the six documented strings c('is','nomis','ccmn','ruv2','ruvrand','ruvrandclust') and rejects other values.
- The returned normalized data matrix has the same samples-by-metabolites dimensions as the input featuredata.
- For nomis/ccmn, qcmets resolves via metabolitedata$IS==1 and ccmn uses the supplied factors (e.g. sampledata$gender).
- For ruv methods, the output includes estimated unwanted-variation factors (W) and k (e.g. k=2 for ruv2 with factormat from gender, Age, bmi).
- Normalized output is assessable with the package's evaluation functions, e.g. LinearModelFit residual RLA plots and p-value distributions.

## Limitations

- Requires the three-part input format (featuredata, metabolitedata, sampledata) with unique sample and metabolite row/column names; metabolitedata must designate internal/external standards.
- Only the six documented method strings are supported by the NormQcmets dispatch; other values are rejected.
- ruv2/ruvrand require an explicit factor matrix and a chosen k; the article demonstrates k=2 for ruv2 and k=1 for ruvrandclust (uv_ruvrandclust).

## Evidence
