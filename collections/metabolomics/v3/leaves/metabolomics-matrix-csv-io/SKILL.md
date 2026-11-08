---
name: metabolomics-matrix-csv-io
description: Use when a metabolomics analysis step needs its input read from or written
  to CSV as a metabolite x sample matrix — e.g. before lilikoi.preproc_norm/preproc_knn
  or after producing normalized/imputed tables.
when_to_use_negative:
- The data is already an in-memory R matrix/data.frame from a prior Lilikoi step and
  only needs to be passed directly to the next module — re-reading CSV adds no value.
- The file is raw mass-spectrometry output (e.g. mzML, NetCDF) rather than a pre-tabulated
  metabolite x sample CSV; Lilikoi v2.0's CSV I/O does not parse spectral files.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lilikoi (Lilikoi v2.0)
  role: Supplies lilikoi.Loaddata for reading the metabolite x sample CSV into dataSet/Metadata,
    and the package's extdata plasma_breast_cancer.csv as the reference input format;
    preprocessing functions write their matrices back to CSV.
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Runtime environment in which lilikoi.Loaddata and the CSV read/write of metabolite
    matrices are executed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:60cbbc076c79f9a093a2ba626075168fab7ea6330ed48d63db555df40c1e1b75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  - lilikoi (Lilikoi v2.0)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-matrix-csv-io

## Summary

Load and round-trip metabolite-by-sample intensity matrices to/from CSV within the Lilikoi v2.0 R environment, using lilikoi.Loaddata to split a plasma_breast_cancer.csv-style file into a dataSet (metabolite rows, sample columns) and Metadata (labels/covariates), so downstream normalization, imputation, and pathway modules receive a consistent matrix structure.

## When to use

Use when a metabolomics analysis step needs its input read from or written to CSV as a metabolite x sample matrix — e.g. before lilikoi.preproc_norm/preproc_knn or after producing normalized/imputed tables.

## When NOT to use

- The data is already an in-memory R matrix/data.frame from a prior Lilikoi step and only needs to be passed directly to the next module — re-reading CSV adds no value.
- The file is raw mass-spectrometry output (e.g. mzML, NetCDF) rather than a pre-tabulated metabolite x sample CSV; Lilikoi v2.0's CSV I/O does not parse spectral files.

## Inputs

- CSV file with a metabolite x sample intensity matrix (e.g. plasma_breast_cancer.csv packaged in lilikoi extdata)
- Optional metadata columns (e.g. Label) parsed by lilikoi.Loaddata into the Metadata object

## Outputs

- R list from lilikoi.Loaddata containing dataSet (metabolite x sample numeric matrix) and Metadata (data frame)
- CSV files of normalized or KNN-imputed metabolite x sample matrices with unchanged metabolite IDs and sample columns

## How to apply

In R, call lilikoi.Loaddata(file=system.file('extdata','plasma_breast_cancer.csv',package='lilikoi')) to read the CSV; the function returns a list with dataSet (the metabolite x sample intensity matrix) and Metadata (sample labels used as measurementLabels later). Preserve metabolite identifiers as the first column/row names and sample IDs as column headers so every downstream Lilikoi module (PDSfun, machine_learning, prognosis) can index samples consistently. When writing results — e.g. after lilikoi.preproc_norm in standard/quantile/median-fold mode or lilikoi.preproc_knn imputation — serialize each resulting matrix to its own CSV file, keeping the original metabolite IDs and sample columns. Before proceeding, verify that each output CSV has the same dimensions as the source plasma_breast_cancer.csv matrix and that quantile-normalized columns share identical empirical distributions.

## Related tools

- **lilikoi (Lilikoi v2.0)** (Supplies lilikoi.Loaddata for reading the metabolite x sample CSV into dataSet/Metadata, and the package's extdata plasma_breast_cancer.csv as the reference input format; preprocessing functions write their matrices back to CSV.) — https://github.com/lanagarmire/lilikoi2
- **R** (Runtime environment in which lilikoi.Loaddata and the CSV read/write of metabolite matrices are executed.)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); dataSet <- dt$dataSet; Metadata <- dt$Metadata
```

## Evaluation signals

- lilikoi.Loaddata returns both dataSet and Metadata; dataSet must have metabolites as rows and samples as columns matching the source CSV.
- Each written CSV has the same metabolite and sample dimensions as the input plasma_breast_cancer.csv matrix.
- Quantile-normalized output columns have identical empirical distributions (preprocessCore normalize.quantiles applied).
- Metadata$Label is preserved so it can serve as measurementLabels in lilikoi.machine_learning.

## Limitations

- Only handles tabulated CSV matrices; the article/README give no spectral-file parsing, so raw instrument data must be converted beforehand.
- The demonstration dataset is plasma_breast_cancer.csv (breast-cancer plasma metabolomics); other matrices must already conform to the same metabolite x sample layout.
- No format coercion is described — matrices that lack explicit metabolite IDs or sample headers may fail downstream indexing by Lilikoi modules.

## Evidence

- [abstract] Lilikoi v2.0 is a modern, comprehensive package to enable metabolomics analysis in R programming environment.: "Lilikoi v2.0 is a modern, comprehensive package to enable metabolomics analysis in R programming environment."
- [intro] Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation): "Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis"
- [readme] README Loaddata example: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
