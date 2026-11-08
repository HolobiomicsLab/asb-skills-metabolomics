---
name: quantile-normalization-of-omics-matrices
description: Use when raw metabolite-sample matrices (e.g., plasma_breast_cancer.csv)
  show sample-to-sample distribution shifts before pathway transformation or machine
  learning.
when_to_use_negative:
- When the goal is z-score standardization or median-fold-change scaling rather than
  distribution alignment — lilikoi.preproc_norm offers 'standard' and 'median-fold'
  modes that are better suited.
- When the matrix already has empirically identical intensity distributions across
  samples (quantile normalization would be redundant and alter within-sample rank
  ordering).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3168
tools:
- name: Lilikoi v2.0
  role: Provides lilikoi.preproc_norm in quantile mode, wrapping preprocessCore's
    normalize.quantiles to equalize metabolite intensity distributions across samples.
  repo: github.com/lanagarmire/lilikoi2
- name: preprocessCore
  role: Supplies the normalize.quantiles function that lilikoi.preproc_norm applies
    to the metabolite-sample matrix.
- name: R
  role: Runtime environment in which Lilikoi v2.0 and preprocessCore are executed
    for normalization.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:447ec05b5d508af78a9f85d7d09867e991fe884ce81539016426633e009bb427
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lilikoi v2.0
  - R
  - preprocessCore
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# quantile-normalization-of-omics-matrices

## Summary

Applies quantile normalization via preprocessCore's normalize.quantiles within Lilikoi v2.0's lilikoi.preproc_norm function to equalize the empirical metabolite intensity distributions across samples in a metabolomics matrix.

## When to use

Use when raw metabolite-sample matrices (e.g., plasma_breast_cancer.csv) show sample-to-sample distribution shifts before pathway transformation or machine learning.

## When NOT to use

- When the goal is z-score standardization or median-fold-change scaling rather than distribution alignment — lilikoi.preproc_norm offers 'standard' and 'median-fold' modes that are better suited.
- When the matrix already has empirically identical intensity distributions across samples (quantile normalization would be redundant and alter within-sample rank ordering).

## Inputs

- Metabolite-by-sample intensity matrix (CSV, e.g. plasma_breast_cancer.csv)
- Metadata / sample labels loaded via lilikoi.Loaddata

## Outputs

- Quantile-normalized metabolite-by-sample matrix (CSV, same dimensions and identifiers as input)

## How to apply

Load the metabolites-by-samples matrix (e.g., the built-in plasma_breast_cancer.csv via lilikoi.Loaddata) and call lilikoi.preproc_norm in quantile mode, which invokes preprocessCore::normalize.quantiles to equalize the empirical intensity distributions across sample columns. The rationale is that raw metabolomics intensities vary in overall distribution between runs or samples, which would bias downstream feature selection and classification; quantile normalization forces all columns onto the same empirical distribution while preserving per-metabolite rank structure. Lilikoi v2.0 additionally offers standard and median-fold normalization modes in the same function, so choose quantile mode when the goal is distribution alignment across samples rather than z-scoring or fold-change scaling. Write the normalized metabolite-sample matrix to CSV, preserving metabolite identifiers and sample columns, before passing it to downstream Lilikoi modules such as lilikoi.PDSfun for pathway transformation or lilikoi.machine_learning for classification.

## Related tools

- **Lilikoi v2.0** (Provides lilikoi.preproc_norm in quantile mode, wrapping preprocessCore's normalize.quantiles to equalize metabolite intensity distributions across samples.) — github.com/lanagarmire/lilikoi2
- **preprocessCore** (Supplies the normalize.quantiles function that lilikoi.preproc_norm applies to the metabolite-sample matrix.)
- **R** (Runtime environment in which Lilikoi v2.0 and preprocessCore are executed for normalization.)

## Examples

```
norm <- lilikoi.preproc_norm(dataSet, mode="quantile")  # uses preprocessCore normalize.quantiles
```

## Evaluation signals

- Each output column's empirical distribution should be identical after quantile normalization — verify by comparing sorted column values across samples.
- The normalized matrix must retain the same metabolite identifiers and sample columns as the input plasma_breast_cancer.csv matrix.
- Output CSV should be written with metabolite identifiers as row labels and sample columns preserved, matching the input dimensions.
- Downstream pathway transformation (lilikoi.PDSfun) should accept the normalized matrix without dimension mismatch errors.

## Limitations

- Quantile normalization forces identical empirical distributions across samples, which can remove real biological variation in overall metabolite abundance; Lilikoi offers standard and median-fold modes as alternatives.
- The article/README does not document the normalization mode default or per-mode parameter details beyond the mode names in lilikoi.preproc_norm.

## Evidence

- [intro] Additional preprocessing methods: "Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation)"
- [readme] preprocessCore quantile normalization: [evidence span withheld — anchor status: not_found]
- [readme] Data loading from built-in dataset: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
- [readme] Workflow step order — preprocessing precedes pathway transformation: "Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
