---
name: reference-vector-normalization
description: Use when metabolomics samples differ in total abundance due to physical
  quantity differences (e.g., tissue mass, sample volume) and a per-sample reference
  vector such as weight or volume is available to normalize against.
when_to_use_negative:
- No meaningful per-sample reference vector (weight/volume) exists — use the sum,
  median, or mean NormScaling variants instead
- Unwanted variation is batch/matrix related rather than a total-abundance scaling
  issue — the package routes those cases to NormQcmets, NormQcsamples (QC-RLSC), or
  NormCombined
- Input is raw peak intensities with missing values or zeros not yet handled — pre-process
  with LogTransform(zerotona=TRUE) and MissingValues first
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NormalizeMets
  role: Provides the NormScaling function whose method='ref' variant performs reference-vector
    scaling normalization of featuredata
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: Execution environment for running NormalizeMets (version 3.4.3 or higher)
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:87b65844e36eb1607a265270f2559e41e8473eb9f2cf3098e2487cf676e2ea8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Reference-vector normalization

## Summary

A scaling normalization from the NormalizeMets package's NormScaling function in which metabolite abundances in each sample are scaled against a user-supplied reference vector, such as sample weight or volume. It is one of four NormScaling methods (sum, median, mean, ref) for correcting sample-specific total abundance differences in metabolomics featuredata.

## When to use

Use when metabolomics samples differ in total abundance due to physical quantity differences (e.g., tissue mass, sample volume) and a per-sample reference vector such as weight or volume is available to normalize against.

## When NOT to use

- No meaningful per-sample reference vector (weight/volume) exists — use the sum, median, or mean NormScaling variants instead
- Unwanted variation is batch/matrix related rather than a total-abundance scaling issue — the package routes those cases to NormQcmets, NormQcsamples (QC-RLSC), or NormCombined
- Input is raw peak intensities with missing values or zeros not yet handled — pre-process with LogTransform(zerotona=TRUE) and MissingValues first

## Inputs

- featuredata matrix (metabolites in columns, unique sample names as row names) in log-transformed, missing-value-imputed form (e.g., imp$featuredata from LogTransform + MissingValues)
- refvec: numeric reference vector (e.g., sample weight or volume) with one value per sample

## Outputs

- NormScaling output object containing the normalized featuredata matrix
- Optional CSV file of the normalized featuredata when saveoutput=TRUE and outputname is set

## How to apply

Prepare the analysis-ready matrix by log-transforming the featuredata with LogTransform(zerotona=TRUE) and imputing missing values with MissingValues(method="knn", feature.cutoff=0.8, sample.cutoff=0.8, k=10), giving imp$featuredata. Then call NormScaling on imp$featuredata with method="ref", passing the per-sample reference values (e.g., sample weight or volume) via the refvec argument. Set saveoutput=TRUE with a named outputname to write the normalized featuredata matrix to disk as a CSV. Rationale: the ref method is the NormScaling variant for external physical references, complementing the sum, median, and mean per-sample scaling variants; the method argument choices are c("median", "mean", "sum", "ref").

## Related tools

- **NormalizeMets** (Provides the NormScaling function whose method='ref' variant performs reference-vector scaling normalization of featuredata) — https://github.com/metabolomicstats/NormalizeMets
- **R** (Execution environment for running NormalizeMets (version 3.4.3 or higher)) — https://cran.r-project.org/

## Examples

```
NormScaling(imp$featuredata, method="ref", refvec=sampleweights, saveoutput=TRUE, outputname="ref_normalized")
```

## Evaluation signals

- NormScaling output object contains a normalized featuredata matrix with the same dimensions as the input
- Row names (unique sample names) and column names (unique metabolite names) of the output matrix match the input featuredata
- With saveoutput=TRUE, the saved CSV (named via outputname) exists and preserves sample/metabolite names
- Per-sample abundances shift proportionally to the supplied refvec values — samples with larger weight/volume are scaled down correspondingly

## Limitations

- Requires a valid refvec; the other NormScaling methods (median, mean, sum) must be used when no reference vector is available
- Only addresses total-abundance scaling; does not correct batch effects or matrix effects, for which the package directs users to QC-based methods
- Excel interface requires Microsoft Excel (2016 or later) on Windows 7 or higher; R interface requires R 3.4.3 or higher

## Evidence

- [methods] NormScaling function provides four scaling normalizations including reference-vector normalization: [evidence span withheld — anchor status: not_found]
- [readme] README on input data format for featuredata: [evidence span withheld — anchor status: external_doc]
