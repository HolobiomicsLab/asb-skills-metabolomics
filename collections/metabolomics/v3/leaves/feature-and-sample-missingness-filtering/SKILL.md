---
name: feature-and-sample-missingness-filtering
description: Use when a metabolomics feature data matrix (peak intensities) contains
  missing values before normalization, and high-missingness features or samples must
  be removed prior to imputation.
when_to_use_negative:
- The data are already imputed or complete (no missing intensities), in which case
  MissingValues would needlessly alter the matrix
- The goal is normalization itself (QC-metabolite, QC-sample RLSC, or scaling) rather
  than the missing-value preprocessing that precedes it
- The input is not a NormalizeMets-format featuredata matrix with sample and metabolite
  annotation dataframes
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NormalizeMets
  role: Provides the MissingValues() function with feature.cutoff/sample.cutoff filtering
    and knn or half-minimum (replace) imputation; runs within R and also via the Excel
    GUI
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which NormalizeMets and its
    MissingValues step are executed
  repo: https://cran.r-project.org/
- name: impute
  role: Underlying imputation dependency supporting the k-nearest neighbour method
    in the MissingValues step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:5ea9c11c16213edd70805e044e5ff9ebd75c0b0428be7dd157d748f8df863ac7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - impute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Feature and sample missingness filtering

## Summary

A preprocessing step from the NormalizeMets workflow that drops metabolite features and samples exceeding a missingness threshold and imputes remaining gaps, using k-th nearest neighbour (knn) or half-minimum replacement.

## When to use

Use when a metabolomics feature data matrix (peak intensities) contains missing values before normalization, and high-missingness features or samples must be removed prior to imputation.

## When NOT to use

- The data are already imputed or complete (no missing intensities), in which case MissingValues would needlessly alter the matrix
- The goal is normalization itself (QC-metabolite, QC-sample RLSC, or scaling) rather than the missing-value preprocessing that precedes it
- The input is not a NormalizeMets-format featuredata matrix with sample and metabolite annotation dataframes

## Inputs

- featuredata: metabolomics data matrix of metabolite peak intensities with unique sample names as row names and unique metabolite names as column names
- sampledata: dataframe of sample-specific information with unique sample names as row names
- metabolitedata: dataframe of metabolite-specific information (e.g. internal/external standards, controls) with metabolite names as row names
- Optionally, log-transformed featuredata from LogTransform(zerotona=TRUE)

## Outputs

- imp$featuredata: the completed feature data matrix with high-missingness features/samples removed and remaining missing values imputed, ready for downstream normalization
- Filtered/imputed matrices retaining samples and features passing the 0.8 cutoffs

## How to apply

Load the metabolomics data in the NormalizeMets three-part format (featuredata matrix of peak intensities with samples as rows and metabolites as columns, plus sampledata and metabolitedata dataframes). Optionally log-transform featuredata with LogTransform(zerotona=TRUE) first. Call MissingValues on the featuredata with feature.cutoff=0.8 and sample.cutoff=0.8: features and samples missing more than these proportions are dropped, so a feature/samples is retained only if present in at least 80% of samples/features. Set method='knn' with k=10 to impute remaining gaps via k-nearest neighbours, or method='replace' to substitute half the minimum observed value. The completed matrix is returned as imp$featuredata and feeds downstream normalization (e.g. NormQcmets, NormScaling).

## Related tools

- **NormalizeMets** (Provides the MissingValues() function with feature.cutoff/sample.cutoff filtering and knn or half-minimum (replace) imputation; runs within R and also via the Excel GUI) — https://github.com/metabolomicstats/NormalizeMets
- **R** (Runtime environment (version 3.4.3 or higher) in which NormalizeMets and its MissingValues step are executed) — https://cran.r-project.org/
- **impute** (Underlying imputation dependency supporting the k-nearest neighbour method in the MissingValues step)

## Examples

```
imp <- MissingValues(logdata$featuredata, sampledata_eg, metabolitedata_eg, feature.cutoff=0.8, sample.cutoff=0.8, method="knn", k=10)
```

## Evaluation signals

- Features and samples retained in imp$featuredata satisfy the cutoffs: each retained feature has non-missing values in ≥80% of samples (feature.cutoff=0.8) and each retained sample in ≥80% of features (sample.cutoff=0.8)
- No missing values remain in imp$featuredata — every gap below the cutoffs is imputed via knn (k=10) or half-minimum replacement
- Row and column names of the output match the original unique sample and metabolite names after filtering
- The output matrix dimensions are consistent with the number of features/samples passing the 0.8 cutoffs applied to the input logdata$featuredata
- Downstream normalization functions (e.g. LinearModelFit, NormQcmets) accept imp$featuredata without missing-value errors

## Limitations

- Choice between method='knn' and method='replace' depends on the nature of the missing data; the paper does not prescribe a single default, so the decision point must be justified per dataset
- MissingValues depends on the NormalizeMets input format: featuredata, sampledata and metabolitedata with matching unique row names; non-conforming inputs will fail
- The step operates on the (optionally log-transformed) feature matrix; zeros may need prior conversion to NA via LogTransform(zerotona=TRUE) before imputation
- The package requires R 3.4.3 or higher (and Microsoft Excel 2016+ for the GUI interface)

## Evidence

- [methods] MissingValues function signature with cutoffs and methods: "MissingValues(featuredata, sampledata, as.dataframe(metabolitedata), feature.cutoff = 0.8, sample.cutoff = 0.8, method = c("knn", "replace"), k = 10,"
- [methods] MissingValues purpose in the workflow: "The following `MissingValues()` function can be used to replace missing values, depending on the nature of missing data."
- [readme] Input data format requirements: [evidence span withheld — anchor status: external_doc]
