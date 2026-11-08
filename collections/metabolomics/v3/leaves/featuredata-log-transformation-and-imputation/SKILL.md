---
name: featuredata-log-transformation-and-imputation
description: Use when raw metabolite peak-intensity data has been arranged into the
  NormalizeMets 'featuredata' format (samples as rows, metabolites as columns) and
  needs log transformation plus missing-value imputation before applying NormQcmets,
  NormScaling, or other normalization functions.
when_to_use_negative:
- Data is already log-transformed and complete — re-running LogTransform with zerotona=TRUE
  would introduce spurious NAs and trigger unnecessary imputation
- The task is QC-sample-based signal correction (NormQcsamples/QC-RLSC) or scaling
  normalization, which the article treats as separate workflow steps applied after
  this preprocessing
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NormalizeMets
  role: provides LogTransform and MissingValues functions used for log transformation
    and knn imputation of featuredata
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: runtime environment (version 3.4.3 or higher) in which the NormalizeMets preprocessing
    functions are executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:63cbd50f1655eaa908b78d0935c3bed5bf16b0b5b7550b0e8eb8c741ba60d7ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# featuredata-log-transformation-and-imputation

## Summary

Preprocesses a metabolomics featuredata matrix for the NormalizeMets workflow by log-transforming peak intensities (converting zeros to NA) and then imputing remaining missing values with k-nearest-neighbour (knn) so downstream normalization methods receive a complete, roughly log-scale data matrix.

## When to use

Use when raw metabolite peak-intensity data has been arranged into the NormalizeMets 'featuredata' format (samples as rows, metabolites as columns) and needs log transformation plus missing-value imputation before applying NormQcmets, NormScaling, or other normalization functions.

## When NOT to use

- Data is already log-transformed and complete — re-running LogTransform with zerotona=TRUE would introduce spurious NAs and trigger unnecessary imputation
- The task is QC-sample-based signal correction (NormQcsamples/QC-RLSC) or scaling normalization, which the article treats as separate workflow steps applied after this preprocessing

## Inputs

- featuredata: samples x metabolites matrix of metabolite peak intensities (or concentrations) with unique sample row names and unique metabolite column names
- sampledata: dataframe of sample-specific information (sample type, analysis order, factors of interest) keyed by sample names
- metabolitedata: dataframe of metabolite-specific information (e.g. internal/external standard designation), keyed by metabolite names

## Outputs

- Log-transformed featuredata matrix (zeros converted to NA via zerotona=TRUE)
- Imputed featuredata matrix with missing values replaced via knn (k=10), with same samples x metabolites dimensions as the input

## How to apply

First ensure the data matrix is in featuredata format with unique sample names as row names and unique metabolite names as column names. Apply LogTransform(featuredata, base=exp(1), zerotona=TRUE, saveoutput=FALSE) so that zero intensities are converted to NA, making them eligible for imputation rather than distorting the log scale. Then run MissingValues(featuredata, sampledata, metabolitedata, feature.cutoff=0.8, sample.cutoff=0.8, method='knn', k=10) to replace missing values; the feature.cutoff and sample.cutoff parameters (0.8 each) restrict imputation to metabolites and samples that retain enough measured values. The rationale, per the article, is that log transformation stabilizes multiplicative variation in peak intensities and that the knn imputation leverages correlation structure among metabolites, producing a complete matrix acceptable to normalization and downstream analyses such as LinearModelFit, PcaPlots, and HeatMap.

## Related tools

- **NormalizeMets** (provides LogTransform and MissingValues functions used for log transformation and knn imputation of featuredata) — https://github.com/metabolomicstats/NormalizeMets
- **R** (runtime environment (version 3.4.3 or higher) in which the NormalizeMets preprocessing functions are executed) — https://cran.r-project.org/

## Examples

```
imp <- MissingValues(LogTransform(featuredata, zerotona=TRUE)$featuredata, sampledata, metabolitedata, feature.cutoff=0.8, sample.cutoff=0.8, method="knn", k=10)
```

## Evaluation signals

- After LogTransform with zerotona=TRUE, the matrix contains no zero intensities; former zeros appear as NA
- After MissingValues with method='knn', the output featuredata contains no NA/missing entries and retains the same samples x metabolites dimensions as the input
- Imputation only proceeds for features and samples within the cutoffs feature.cutoff=0.8 and sample.cutoff=0.8
- Transformed values are on the natural-log scale (base=exp(1)), consistent with subsequent visualization functions such as RlaPlots and PcaPlots

## Limitations

- MissingValues offers only 'knn' and 'replace' methods, and knn imputation quality depends on enough correlated metabolites passing the 0.8 feature/sample cutoffs
- zerotona=TRUE converts true zero intensities to NA, so imputation replaces them with estimated rather than observed values
- Input must strictly follow the three-part NormalizeMets format (featuredata, metabolitedata, sampledata) with unique row names in each

## Evidence

- [methods] LogTransform with zerotona: "LogTransform <- function(featuredata, base=exp(1), saveoutput=FALSE,
    outputname="log.results",zerotona=FALSE)"
- [methods] MissingValues signature: "MissingValues(featuredata, sampledata, as.dataframe(metabolitedata), feature.cutoff = 0.8,
  sample.cutoff = 0.8, method = c("knn", "replace"), k = 10,"
- [readme] featuredata format requirement: [evidence span withheld — anchor status: external_doc]
- [methods] log transform workflow step: "A metabolomics data matrix in the _featuredata_ format can be transformed using the following function. LogTransform"
