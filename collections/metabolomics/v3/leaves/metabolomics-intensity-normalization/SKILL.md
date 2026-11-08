---
name: metabolomics-intensity-normalization
description: Use when raw metabolite-by-sample intensity data (e.g. plasma_breast_cancer.csv)
  requires normalization or imputation prior to pathway scoring or classification.
when_to_use_negative:
- Input is already a pathway-level feature matrix (PDSmatrix) rather than raw metabolite
  intensities — use pathway transformation instead.
- The goal is disease classification or prognosis modeling on already-normalized data
  rather than preparing intensities for downstream analysis.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lilikoi.preproc_norm
  role: Performs standard, quantile, or median-fold normalization on the metabolite-sample
    intensity matrix.
  repo: https://github.com/lanagarmire/lilikoi2
- name: lilikoi.preproc_knn
  role: Imputes missing metabolite intensity values using k-nearest-neighbor imputation
    (impute.knn).
  repo: https://github.com/lanagarmire/lilikoi2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:eadaafc3d8587feb602df3890859bedb847f0342fbf6059fcd7ec64f26333755
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  - lilikoi.preproc_knn
  - lilikoi.preproc_norm
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-intensity-normalization

## Summary

Normalizes metabolomics intensity matrices using Lilikoi v2.0's lilikoi.preproc_norm, which offers standard, quantile, and median-fold normalization for metabolite-by-sample data, plus optional knn imputation via lilikoi.preproc_knn.

## When to use

Use when raw metabolite-by-sample intensity data (e.g. plasma_breast_cancer.csv) requires normalization or imputation prior to pathway scoring or classification.

## When NOT to use

- Input is already a pathway-level feature matrix (PDSmatrix) rather than raw metabolite intensities — use pathway transformation instead.
- The goal is disease classification or prognosis modeling on already-normalized data rather than preparing intensities for downstream analysis.

## Inputs

- metabolite-by-sample intensity matrix (e.g. plasma_breast_cancer.csv)

## Outputs

- normalized metabolite-sample intensity matrix (standard, quantile, or median-fold)
- imputed metabolite-sample matrix with missing values filled by knn

## How to apply

Load the metabolite-by-sample matrix into R (e.g. lilikoi.Loaddata on the built-in plasma_breast_cancer.csv). Call lilikoi.preproc_norm choosing standard, quantile (via preprocessCore normalize.quantiles), or median-fold normalization depending on the intended downstream analysis. If missing intensity values are present, run lilikoi.preproc_knn to impute them using the k-nearest-neighbor impute.knn algorithm. Validate that the resulting matrix retains the original metabolite and sample dimensions, and for quantile mode that each column's empirical intensity distribution is equalized.

## Related tools

- **lilikoi.preproc_norm** (Performs standard, quantile, or median-fold normalization on the metabolite-sample intensity matrix.) — https://github.com/lanagarmire/lilikoi2
- **lilikoi.preproc_knn** (Imputes missing metabolite intensity values using k-nearest-neighbor imputation (impute.knn).) — https://github.com/lanagarmire/lilikoi2

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); normalized <- lilikoi.preproc_norm(dt$dataSet, method="quantile")
```

## Evaluation signals

- Output matrix must retain the same number of metabolite rows and sample columns as the input matrix.
- Quantile-normalized output columns should have identical empirical intensity distributions across samples.
- After lilikoi.preproc_knn, no missing/NA intensity values should remain in the imputed matrix.
- Normalization choice must match the intended downstream step (pathway transformation, feature selection, or classification).

## Limitations

- Quantile normalization relies on preprocessCore's normalize.quantiles; results depend on sample distribution assumptions.
- Knn imputation is the package default for missing values; alternative imputation strategies are not described in the README.
- Normalization must be applied before feature mapping to HMDB IDs and pathway transformation; applying it after may change downstream pathway scores.

## Evidence

- [abstract] Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression.: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [intro] include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation): "we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation)"
- [readme] Load the built-in plasma_breast_cancer.csv metabolomics dataset: "lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
