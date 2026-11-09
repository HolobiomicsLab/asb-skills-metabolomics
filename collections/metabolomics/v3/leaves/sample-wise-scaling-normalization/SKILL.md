---
name: sample-wise-scaling-normalization
description: Use when a log-transformed, missing-value-imputed metabolomics featuredata
  matrix (samples as rows, metabolites as columns) needs sample-wise scaling normalization,
  or when one of sum, median, mean, or reference-vector normalization is required.
when_to_use_negative:
- When QC-sample-based signal correction is needed instead of simple scaling — use
  NormQcsamples (QC-RLSC) rather than NormScaling.
- When normalization should exploit internal/external standards or quality control
  metabolites — use NormQcmets methods (is, nomis, ccmn, ruv2, ruvrand, ruvrandclust).
- When a sequential combination of normalizations is the goal, call NormCombined directly
  rather than chaining NormScaling manually (though NormCombined itself chains median
  scaling after rlsc by default).
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NormalizeMets
  role: R package providing the NormScaling (and NormCombined) functions that implement
    the sum, median, mean, and ref scaling normalizations
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: software environment in which the NormalizeMets package is run (version 3.4.3
    or higher)
  repo: https://cran.r-project.org/
- name: statTarget
  role: reference implementation of the QC-RLSC signal correction that NormCombined
    chains before median scaling
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:e98f4becc37d1d6e6f9f3cd9f9ebfc62801dd8efb81c40d9f5a708e050d89ae5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sample-wise-scaling-normalization

## Summary

Scaling normalization of metabolomics feature data via the NormalizeMets NormScaling function, which normalizes each sample by its total sum, median, or mean peak intensity (or concentration), or to a user-supplied reference vector such as sample weight or volume.

## When to use

Use when a log-transformed, missing-value-imputed metabolomics featuredata matrix (samples as rows, metabolites as columns) needs sample-wise scaling normalization, or when one of sum, median, mean, or reference-vector normalization is required.

## When NOT to use

- When QC-sample-based signal correction is needed instead of simple scaling — use NormQcsamples (QC-RLSC) rather than NormScaling.
- When normalization should exploit internal/external standards or quality control metabolites — use NormQcmets methods (is, nomis, ccmn, ruv2, ruvrand, ruvrandclust).
- When a sequential combination of normalizations is the goal, call NormCombined directly rather than chaining NormScaling manually (though NormCombined itself chains median scaling after rlsc by default).

## Inputs

- featuredata matrix of metabolite peak intensities (or concentrations) with unique sample names as row names and unique metabolite names as column names
- log-transformed and imputed featuredata (e.g. imp$featuredata from LogTransform followed by MissingValues)
- refvec reference vector (e.g. sample weight or volume) for method='ref'

## Outputs

- normalized featuredata matrix with the same samples (rows) and metabolites (columns) as the input
- CSV file (when saveoutput=TRUE with outputname set)

## How to apply

Prepare the analysis-ready matrix by log-transforming with LogTransform(zerotona=TRUE) and imputing missing values with MissingValues(method='knn', feature.cutoff=0.8, sample.cutoff=0.8, k=10); pass imp$featuredata to NormScaling. Choose method='median', 'mean', or 'sum' to divide each sample by its median, mean, or total peak intensity (or concentration), or method='ref' with refvec to normalize to a reference vector such as sample weight or volume. Set saveoutput=TRUE with a named outputname to write the normalized featuredata matrix to disk as a CSV. The function signature is NormScaling(featuredata, method = c('median', 'mean', 'sum', 'ref'), refvec = NULL, saveoutput = FALSE, outputname = NULL, ...). For a combined workflow, NormCombined chains rlsc signal correction followed by median scaling by default.

## Related tools

- **NormalizeMets** (R package providing the NormScaling (and NormCombined) functions that implement the sum, median, mean, and ref scaling normalizations) — https://github.com/metabolomicstats/NormalizeMets
- **R** (software environment in which the NormalizeMets package is run (version 3.4.3 or higher)) — https://cran.r-project.org/
- **statTarget** (reference implementation of the QC-RLSC signal correction that NormCombined chains before median scaling)

## Examples

```
NormScaling(imp$featuredata, method="median", saveoutput=TRUE, outputname="median_scaling_results")
```

## Evaluation signals

- The normalized featuredata matrix is returned in the NormScaling output object with the same row names (samples) and column names (metabolites) as the input feature matrix.
- When saveoutput=TRUE with outputname set, the saved CSV exists and matches the input dimensions and naming.
- Normalized values differ from the raw unadjusted abundances, confirming the scaling stage was applied.
- For method='ref', normalization used the supplied refvec reference vector (e.g. sample weight or volume).
- For NormCombined output, both stages (rlsc correction then median scaling) are reflected in the final values, saved as e.g. normcombined_results.csv.

## Limitations

- The ref method requires a reference vector (refvec); calling method='ref' without one is invalid.
- Scaling normalization is one of several options; the package exists because the best method depends on the dataset and research question, so comparative evaluation against other NormalizeMets methods is advised.
- Missing values must be handled first (e.g. MissingValues with method='knn', feature.cutoff=0.8, sample.cutoff=0.8, k=10); features/samples below the cutoffs should be removed.

## Evidence

- [methods] NormScaling provides four scaling normalizations: [evidence span withheld — anchor status: not_found]
- [readme] input data format: [evidence span withheld — anchor status: external_doc]
- [readme] requirements: [evidence span withheld — anchor status: external_doc]
