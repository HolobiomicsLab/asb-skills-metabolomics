---
name: missing-value-knn-imputation
description: Use when a metabolomics metabolite-by-sample matrix (e.g., plasma_breast_cancer.csv
  from Lilikoi) contains missing intensity values that must be filled before pathway
  transformation, feature selection, or machine learning classification.
when_to_use_negative:
- Input matrix already contains complete data with no missing values — imputation
  is unnecessary and adds noise
- Analysis goal is something other than preprocessing a metabolomics matrix, e.g.,
  prognosis modeling (lilikoi.prognosis) or KEGG visualization (lilikoi.KEGGplot),
  which are separate Lilikoi v2.0 modules
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Lilikoi v2.0
  role: R package providing lilikoi.preproc_knn, which wraps impute.knn to impute
    missing metabolomics values
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: programming environment in which the Lilikoi v2.0 preprocessing and imputation
    functions are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:dd97b18d4a29ad505dab5f3f925bd6af5216bca44d3166e6fbb32039df39c06c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# missing-value-knn-imputation

## Summary

Imputes missing metabolite intensities in metabolomics matrices using k-nearest neighbors (impute.knn) via Lilikoi v2.0's lilikoi.preproc_knn function, a preprocessing capability added in Lilikoi v2.0 to ready metabolite-sample matrices for downstream classification.

## When to use

Use when a metabolomics metabolite-by-sample matrix (e.g., plasma_breast_cancer.csv from Lilikoi) contains missing intensity values that must be filled before pathway transformation, feature selection, or machine learning classification.

## When NOT to use

- Input matrix already contains complete data with no missing values — imputation is unnecessary and adds noise
- Analysis goal is something other than preprocessing a metabolomics matrix, e.g., prognosis modeling (lilikoi.prognosis) or KEGG visualization (lilikoi.KEGGplot), which are separate Lilikoi v2.0 modules

## Inputs

- metabolite-by-sample intensity matrix with missing values (CSV, e.g., Lilikoi's built-in plasma_breast_cancer.csv)
- Lilikoi v2.0 R package (lilikoi.preproc_knn function)

## Outputs

- complete metabolite-by-sample matrix with missing values imputed (writable to CSV, preserving metabolite identifiers and sample columns)

## How to apply

Load the metabolite-by-sample matrix (e.g., the built-in plasma_breast_cancer.csv shipped with the lilikoi package, via lilikoi.Loaddata) and run lilikoi.preproc_knn, which applies the impute.knn k-nearest-neighbor algorithm to fill missing metabolite intensity values and returns a complete metabolite-sample matrix. This imputation step is one of Lilikoi v2.0's additional preprocessing methods (alongside standard, quantile via preprocessCore normalize.quantiles, and median-fold normalization exposed through lilikoi.preproc_norm) and should be applied before pathway-based feature transformation (lilikoi.PDSfun) and classification (lilikoi.machine_learning), since those modules require complete matrices. Validate that the imputed matrix retains the exact metabolite identifiers and sample columns of the input, with no residual missing values.

## Related tools

- **Lilikoi v2.0** (R package providing lilikoi.preproc_knn, which wraps impute.knn to impute missing metabolomics values) — https://github.com/lanagarmire/lilikoi2
- **R** (programming environment in which the Lilikoi v2.0 preprocessing and imputation functions are executed)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); imputed <- lilikoi.preproc_knn(dt$dataSet)
```

## Evaluation signals

- Imputed output matrix has identical metabolite identifiers and sample columns/dimensions as the input matrix
- No missing (NA) values remain in the imputed metabolite-sample matrix
- Matrix written to CSV preserves the input's row/column ordering so it can be verified by diff against the original
- Imputed values fall within plausible intensity ranges of observed metabolite values in the input matrix

## Limitations

- Imputation is applied as a default for missing metabolomics values within Lilikoi v2.0's preprocessing module; its behavior outside metabolomics matrices is not covered by the article
- Detailed parameter documentation (e.g., number of neighbors k) is not given in the article text or README; consult the Lilikoi2 User Guide for tuning

## Evidence

- [intro] preprocessing methods including knn imputation in Lilikoi v2.0: "Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis"
- [abstract] Lilikoi v2.0 supports data preprocessing as a module: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [readme] README confirms Lilikoi v2 preprocessing support and R environment: "Additionally, Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
- [readme] README shows how to load the built-in plasma_breast_cancer.csv dataset: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
