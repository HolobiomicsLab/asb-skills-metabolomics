---
name: plsda-cross-validated-q2y-estimation
description: Use when evaluating supervised classification of a phenotype (e.g., breast
  cancer status) from metabolomics and/or gene-expression feature tables, and when
  a cross-validated Q2Y metric is needed to compare feature-filtering strategies.
when_to_use_negative:
- The goal is phenotype-affected gene-metabolite interaction testing via linear models
  (IntLIM-style FDR-adjusted interaction p-values), not PLS-DA classification performance
- The input data are raw instrument files (.raw, .d, mzXML) that have not passed SMART's
  data-import, preprocessing, and quality-control modules
- The dataset has no supervised class labels, since PLS-DA is a classification method
  and Q2Y is not defined for purely unsupervised analyses
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: SMART 2.0
  role: Provides the PLS/PLS-DA classification module used to fit the metabolites-only,
    integrated, and dual-filter models and to compute cross-validated Q2Y/R2Y
  repo: https://github.com/YuJenL/SMART
- name: IntLIM
  role: Source of the breast cancer gene-expression and metabolomics example data
    files used as PLS-DA inputs
  repo: https://github.com/Mathelab/IntLIM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:27d8046cdabf1ce6c15e79a1f3b6665c3cfdea0ef3dc37d695ad2cf645a2b8b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - IntLIM
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# plsda-cross-validated-q2y-estimation

## Summary

Fits PLS/PLS-DA classification models in the SMART 2.0 R environment and reports cross-validated Q2Y (and R2Y) performance, as demonstrated on the breast cancer gene-expression (20,254 genes) and metabolomics (504 metabolites) data where the dual-filter model reached Q2Y = 0.701.

## When to use

Use when evaluating supervised classification of a phenotype (e.g., breast cancer status) from metabolomics and/or gene-expression feature tables, and when a cross-validated Q2Y metric is needed to compare feature-filtering strategies.

## When NOT to use

- The goal is phenotype-affected gene-metabolite interaction testing via linear models (IntLIM-style FDR-adjusted interaction p-values), not PLS-DA classification performance
- The input data are raw instrument files (.raw, .d, mzXML) that have not passed SMART's data-import, preprocessing, and quality-control modules
- The dataset has no supervised class labels, since PLS-DA is a classification method and Q2Y is not defined for purely unsupervised analyses

## Inputs

- Gene-expression matrix for the breast cancer cohort (20,254 genes) from Mathelab/IntLIM example data
- Metabolomics feature matrix (504 metabolites) from the same cohort
- Phenotype class labels (cancer/normal) used as the PLS-DA response variable
- Optional covariate table for the covariate-adjusted model

## Outputs

- Cross-validated Q2Y values for three models: metabolites-only (target 0.454), integrated genes+metabolites (target 0.55), dual-filter pFDR<0.05 + top-10 VIP (target 0.701)
- R2Y for the optimized model (target 0.725)
- VIP-ranked feature lists for each fitted model
- plsda_results.csv containing the Q2Y metrics and feature rankings

## How to apply

Load the breast cancer gene-expression and metabolomics tables from the Mathelab/IntLIM repository into R and invoke the SMART 2.0 PLS/PLS-DA statistical-analysis module. Establish baselines first: fit a PLS-DA model on metabolites only (reference Q2Y = 0.454) and an integrated genes+metabolites model (reference Q2Y = 0.55). Then apply the dual-filter strategy — retain features with pFDR < 0.05, rank by VIP, keep the top 10 — refit a covariate-adjusted PLS/PLS-DA model, and compute cross-validated Q2Y and R2Y. The rationale is that combining statistical-significance filtering (pFDR < 0.05) with VIP-based dimensionality reduction maximizes predictive accuracy over unfiltered or single-filter models. Report the three Q2Y values plus VIP-ranked feature lists to plsda_results.csv; the dual-filter model should reproduce Q2Y = 0.701 and R2Y = 0.725.

## Related tools

- **SMART 2.0** (Provides the PLS/PLS-DA classification module used to fit the metabolites-only, integrated, and dual-filter models and to compute cross-validated Q2Y/R2Y) — https://github.com/YuJenL/SMART
- **IntLIM** (Source of the breast cancer gene-expression and metabolomics example data files used as PLS-DA inputs) — https://github.com/Mathelab/IntLIM

## Examples

```
devtools::install_github("YuJenL/SMART"); library(SMART)  # invoke the PLS/PLS-DA module on the IntLIM breast cancer gene/metabolite tables and write plsda_results.csv
```

## Evaluation signals

- Metabolites-only PLS-DA cross-validated Q2Y ≈ 0.454
- Integrated genes+metabolites PLS-DA cross-validated Q2Y ≈ 0.55
- Dual-filter (pFDR < 0.05 + top-10 VIP) covariate-adjusted model attains Q2Y ≈ 0.701 and R2Y ≈ 0.725, exceeding both baseline models
- plsda_results.csv contains exactly the three Q2Y values plus VIP-ranked feature lists
- VIP list of the final model is restricted to the top-10 cutoff, confirming the dual filter was applied

## Limitations

- Q2Y targets (0.454, 0.55, 0.701) are reference values from the article's breast cancer demonstration; other datasets will yield different metrics
- The Mathelab/IntLIM repository is deprecated — the data source has moved to ncats/IntLIM, so download paths may change
- SMART 2.0's peak-identification limitations (e.g., compounds absent from HMDB) are unrelated but indicate module-specific gaps; the PLS-DA module depends on correctly preprocessed/QC-passed feature tables

## Evidence

- [other] dual-filter model performance: [evidence span withheld — anchor status: not_found]
- [other] workflow baseline and final steps: [evidence span withheld — anchor status: not_found]
- [abstract] SMART 2.0 classification module: "introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
- [readme] README module listing: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] breast cancer data availability via IntLIM: [evidence span withheld — anchor status: external_doc]
