---
name: missing-value-imputation-knn
description: Use when a metabolomics featuredata matrix (log-transformed peak intensities)
  still contains missing values that must be filled before normalization functions
  such as NormScaling can be run.
when_to_use_negative:
- The input is already a complete, imputed feature matrix ready for normalization
  (proceed directly to NormScaling or NormQcmets).
- Zeros represent true below-detection-limit observations to be converted to NA by
  LogTransform(zerotona=TRUE) — that conversion belongs to the LogTransform step,
  not to imputation.
- Features or samples exceed the missingness thresholds (more than 20% missing under
  the 0.8 cutoffs), in which case they are dropped rather than imputed and imputation
  alone would be the wrong remedy.
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NormalizeMets
  role: R package providing the MissingValues() function with method='knn' (k=10)
    or 'replace' (half-minimum) and the feature.cutoff/sample.cutoff=0.8 filters
  repo: metabolomicstats/NormalizeMets
- name: impute
  role: R package implementing the k-nearest neighbour algorithm used by MissingValues
    for knn imputation
- name: R
  role: runtime environment (version 3.4.3 or higher) in which the NormalizeMets workflow
    is executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:7076becd7df1b0ff7e687c6a955678dfcdead7cf39a36a5fbb1153707e63a01e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - impute
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# missing-value-imputation-knn

## Summary

Handles missing metabolite intensities in a NormalizeMets featuredata matrix by dropping high-missingness features/samples (feature.cutoff=0.8, sample.cutoff=0.8) and imputing remaining gaps with the k-th nearest neighbour algorithm (k=10), with half-minimum replacement as the alternative 'replace' method.

## When to use

Use when a metabolomics featuredata matrix (log-transformed peak intensities) still contains missing values that must be filled before normalization functions such as NormScaling can be run.

## When NOT to use

- The input is already a complete, imputed feature matrix ready for normalization (proceed directly to NormScaling or NormQcmets).
- Zeros represent true below-detection-limit observations to be converted to NA by LogTransform(zerotona=TRUE) — that conversion belongs to the LogTransform step, not to imputation.
- Features or samples exceed the missingness thresholds (more than 20% missing under the 0.8 cutoffs), in which case they are dropped rather than imputed and imputation alone would be the wrong remedy.

## Inputs

- log-transformed featuredata matrix (metabolites in columns, unique sample names as row names) from LogTransform output
- sampledata dataframe with unique sample names as row names
- metabolitedata dataframe with metabolite names as row names (internal/external standards, positive/negative controls)

## Outputs

- completed featuredata matrix (imp$featuredata) with imputed values and high-missingness features/samples removed
- MissingValues output object carrying the filtered feature matrix for downstream normalization

## How to apply

Load the alldata_eg example dataset (or your own data) and extract the featuredata, sampledata, and metabolitedata components, then log-transform featuredata with LogTransform(zerotona=TRUE) to obtain logdata. Call MissingValues on logdata$featuredata together with sampledata and metabolitedata, setting feature.cutoff=0.8 and sample.cutoff=0.8 so that features or samples missing more than 20% of values are dropped, and method='knn' with k=10 so remaining gaps are imputed by k-nearest neighbours. Choose method='replace' instead when the appropriate model is replacement by half the minimum observed intensity. The rationale is that normalization methods in NormalizeMets require a complete, analysis-ready matrix, and the vignette pipeline feeds imp$featuredata directly into downstream steps such as NormScaling.

## Related tools

- **NormalizeMets** (R package providing the MissingValues() function with method='knn' (k=10) or 'replace' (half-minimum) and the feature.cutoff/sample.cutoff=0.8 filters) — metabolomicstats/NormalizeMets
- **impute** (R package implementing the k-nearest neighbour algorithm used by MissingValues for knn imputation)
- **R** (runtime environment (version 3.4.3 or higher) in which the NormalizeMets workflow is executed) — https://cran.r-project.org/

## Examples

```
imp <- MissingValues(logdata$featuredata, sampledata_eg, metabolitedata_eg, feature.cutoff=0.8, sample.cutoff=0.8, method="knn", k=10)
```

## Evaluation signals

- The returned object exposes imp$featuredata with no remaining NA values after imputation.
- Features and samples with more than 20% missing values (i.e., failing the 0.8 cutoffs) have been removed from the output matrix.
- Row names of the output featuredata are the surviving unique sample names and column names are the surviving metabolite names from the input.
- The matrix dimensions are reduced consistently for rows and columns relative to the pre-filter input.
- Downstream steps (e.g., NormScaling with method='median') accept the matrix without missing-value errors.

## Limitations

- The knn imputation quality depends on the chosen k (default 10); the alternative 'replace' method fills gaps with half the minimum intensity, which may bias low-abundance metabolites.
- Imputation only handles gaps that survive the 0.8 feature/sample cutoffs — highly sparse features or samples are discarded instead of imputed.
- The function operates on the three-part NormalizeMets input format (featuredata, metabolitedata, sampledata); data must be reshaped with unique row names before use.

## Evidence

- [results] knn and half-minimum are the two supported imputation methods: [evidence span withheld — anchor status: not_found]
