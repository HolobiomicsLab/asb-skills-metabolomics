---
name: metabolomics-data-matrix-preprocessing
description: Use when raw metabolomics peak-intensity data must be prepared for normalization
  — e.g. log-transforming featuredata with zeros converted to NA, then filtering and
  imputing missing values before applying NormQcmets/NormQcsamples.
when_to_use_negative:
- Input is already log-transformed and complete — re-running LogTransform(zerotona=TRUE)
  would corrupt values and MissingValues filtering is unnecessary
- The goal is normalization method comparison itself (use NormalizeMets's NormQcmets/NormScaling/NormCombined
  functions instead of preprocessing)
- Working in the Excel GUI-only workflow where the R-level LogTransform/MissingValues
  function signatures do not apply directly
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: NormalizeMets
  role: R package supplying the LogTransform and MissingValues functions used for
    log-transformation, missingness filtering, and knn/half-minimum imputation
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: Software environment required (version 3.4.3 or higher) in which the preprocessing
    functions are executed
  repo: https://cran.r-project.org/
- name: RStudio
  role: Recommended IDE for running the NormalizeMets preprocessing workflow in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:cbd73c5cca28cd4d75d250305f2a76501b674a61f45f8d895a4b7ba9610f8cff
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - RStudio
  - impute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# metabolomics-data-matrix-preprocessing

## Summary

Preprocess a metabolomics feature data matrix in R using the NormalizeMets package: log-transform peak intensities (converting zeros to NA) and handle missing values via k-nearest-neighbour imputation or half-minimum replacement, with feature/sample missingness cutoffs, before downstream normalization.

## When to use

Use when raw metabolomics peak-intensity data must be prepared for normalization — e.g. log-transforming featuredata with zeros converted to NA, then filtering and imputing missing values before applying NormQcmets/NormQcsamples.

## When NOT to use

- Input is already log-transformed and complete — re-running LogTransform(zerotona=TRUE) would corrupt values and MissingValues filtering is unnecessary
- The goal is normalization method comparison itself (use NormalizeMets's NormQcmets/NormScaling/NormCombined functions instead of preprocessing)
- Working in the Excel GUI-only workflow where the R-level LogTransform/MissingValues function signatures do not apply directly

## Inputs

- featuredata: metabolomics data matrix of metabolite peak intensities (or concentrations), unique sample names as row names, unique metabolite names as column names
- sampledata: dataframe of sample-specific information (sample type, order of analysis, factors of interest) keyed by sample names as row names
- metabolitedata: dataframe of metabolite-specific information (internal/external standards, positive/negative controls) keyed by metabolite names as row names

## Outputs

- logdata: log-transformed feature data matrix with zeros converted to NA
- imp$featuredata: completed feature data matrix after missingness filtering and knn (k=10) or half-minimum imputation, ready for normalization

## How to apply

Assemble the three-part input format from the README: featuredata (samples as row names, metabolites as column names), metabolitedata (metabolite-specific metadata such as internal/external standards or positive/negative controls), and sampledata (sample type, analysis order, factors of interest). First run LogTransform on featuredata with base=exp(1) and zerotona=TRUE to obtain logdata, converting zero intensities to NA. Then run MissingValues on logdata$featuredata with sampledata and metabolitedata, setting feature.cutoff=0.8 and sample.cutoff=0.8 to drop features and samples exceeding 80% missingness, and choose method='knn' with k=10 for k-nearest-neighbour imputation or method='replace' for replacement by half the minimum, depending on the nature of the missing data. The completed matrix imp$featuredata is then ready for downstream normalization (e.g. NormQcmets or NormQcsamples) within NormalizeMets, which handles data locally for reproducibility.

## Related tools

- **NormalizeMets** (R package supplying the LogTransform and MissingValues functions used for log-transformation, missingness filtering, and knn/half-minimum imputation) — https://github.com/metabolomicstats/NormalizeMets
- **R** (Software environment required (version 3.4.3 or higher) in which the preprocessing functions are executed) — https://cran.r-project.org/
- **RStudio** (Recommended IDE for running the NormalizeMets preprocessing workflow in R)

## Examples

```
logdata <- LogTransform(featuredata, zerotona=TRUE); imp <- MissingValues(logdata$featuredata, sampledata_eg, as.dataframe(metabolitedata_eg), feature.cutoff=0.8, sample.cutoff=0.8, method="knn", k=10)
```

## Evaluation signals

- After LogTransform(zerotona=TRUE), no zero intensities remain in logdata$featuredata (they appear as NA)
- Features and samples with missingness above feature.cutoff=0.8 / sample.cutoff=0.8 are dropped before imputation
- After method='knn', k=10, the returned imp$featuredata contains no NA values and retains original row (sample) and column (metabolite) names
- Dimensions of imp$featuredata reflect the expected removal of high-missingness features/samples relative to the input featuredata
- The workflow is reproducible locally: the saved code can be re-run to regenerate identical logdata and imp$featuredata

## Limitations

- The imputation choice depends on the nature of missing data — knn (k=10) vs half-minimum replacement must be selected by the analyst, and an inappropriate choice can bias downstream normalization
- Cutoffs of 0.8 for both feature.cutoff and sample.cutoff are defaults; datasets with widespread missingness may lose many features/samples
- LogTransform's zerotona=FALSE default would leave zeros untouched; failing to set it to TRUE means zeros are treated as measured intensities
- Requires R 3.4.3 or higher (and Excel 2016+ if using the GUI interface)

## Evidence

- [readme] Input data format description from README: [evidence span withheld — anchor status: external_doc]
