---
name: log-transformation-of-feature-abundances
description: Use when a raw metabolomics data matrix in the featuredata format (unique
  sample names as row names, unique metabolite names as column names) has skewed positive
  intensities needing log transformation, or when a NormalizeMets normalization call
  such as NormQcsamples should log-transform its output.
when_to_use_negative:
- The input matrix is already log-transformed — a second LogTransform call would double-transform
  the data.
- The data contain zeros or missing values that have not been handled with the MissingValues
  function (default zerotona=FALSE does not convert zeros to NA, and log(0) is undefined).
- The goal is a normalization method comparison on raw-scale data where log transformation
  would confound the evaluation criteria.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: NormalizeMets
  role: Provides the LogTransform function and the lg parameter of NormQcsamples that
    perform the log transformation of featuredata
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: Runtime environment in which NormalizeMets is installed and LogTransform/NormQcsamples
    are executed (version 3.4.3 or higher required)
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:eff6a844211f248582252c952e0f977b095aca5d456a0a8278fc612187c22433
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# log-transformation-of-feature-abundances

## Summary

Log-transforms a metabolomics featuredata matrix (metabolite peak intensities/concentrations) using the NormalizeMets LogTransform function, either as a standalone step or as the lg=TRUE log stage applied after normalization (e.g. QC-RLSC signal correction via NormQcsamples).

## When to use

Use when a raw metabolomics data matrix in the featuredata format (unique sample names as row names, unique metabolite names as column names) has skewed positive intensities needing log transformation, or when a NormalizeMets normalization call such as NormQcsamples should log-transform its output (lg=TRUE).

## When NOT to use

- The input matrix is already log-transformed — a second LogTransform call would double-transform the data.
- The data contain zeros or missing values that have not been handled with the MissingValues function (default zerotona=FALSE does not convert zeros to NA, and log(0) is undefined).
- The goal is a normalization method comparison on raw-scale data where log transformation would confound the evaluation criteria.

## Inputs

- featuredata: metabolomics data matrix of metabolite peak intensities (or concentrations) with unique sample names as row names and unique metabolite names as column names
- sampledata dataframe with sample-specific information (e.g. batch, class, order columns) when used within NormQcsamples

## Outputs

- log-transformed featuredata matrix of the same dimensions (samples x metabolites) as the input
- CSV output (e.g. normalized_qcsamples.csv or outputname="log.results") when saveoutput=TRUE

## How to apply

Call LogTransform on the featuredata matrix with base=exp(1) (natural log), saveoutput=FALSE, outputname="log.results", and zerotona=FALSE by default. In the NormQcsamples(method='rlsc') QC-RLSC workflow, leave lg=TRUE so the function log-transforms the signal-corrected abundances after robust LOESS correction (span=0, deg=2) — the package applies normalization BEFORE log transformation by design. The input featuredata must have unique sample names as row names and unique metabolite names as column names, per the package README input format. If zeros or missing values are present, handle them first with MissingValues(method='knn'/'replace', feature.cutoff=0.8, sample.cutoff=0.8) rather than silently transforming them.

## Related tools

- **NormalizeMets** (Provides the LogTransform function and the lg parameter of NormQcsamples that perform the log transformation of featuredata) — https://github.com/metabolomicstats/NormalizeMets
- **R** (Runtime environment in which NormalizeMets is installed and LogTransform/NormQcsamples are executed (version 3.4.3 or higher required)) — https://cran.r-project.org/

## Examples

```
LogTransform(featuredata, base=exp(1), saveoutput=TRUE, outputname="log.results")
```

## Evaluation signals

- Output matrix has the same number of rows (samples) and columns (metabolites) as the input featuredata, with row and column names preserved.
- All transformed values are present (non-missing) and finite after transformation.
- When invoked via NormQcsamples with lg=TRUE, the saved matrix (e.g. normalized_qcsamples.csv) contains log-scale values derived from the rlsc-corrected abundances, not raw intensities.
- With saveoutput=TRUE, the output file matching outputname (default "log.results") is produced.
- Sanity check on a test cell: output equals log(input) under base=exp(1) for any strictly positive input entry.

## Limitations

- Default zerotona=FALSE means zeros in featuredata are not converted to NA before transformation; users must decide whether to set zerotona=TRUE or pre-treat zeros via MissingValues.
- In NormQcsamples the default lg=TRUE means normalization is applied before log transformation — reordering these steps changes the result and deviates from the package design.
- LogTransform is a single step in the NormalizeMets pipeline; downstream visualization (RlaPlots, PcaPlots, HeatMap) and statistical analyses expect the log-transformed matrix, so skipping the step breaks the documented workflow.
- The package requires R version 3.4.3 or higher.

## Evidence

- [methods] Log stage of QC-RLSC normalization: [evidence span withheld — anchor status: not_found]
- [readme] Input data format: [evidence span withheld — anchor status: external_doc]
