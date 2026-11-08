---
name: vip-feature-filtering-for-pls-models
description: Use when a PLS/PLS-DA classifier on metabolomics or integrated gene-expression
  + metabolomics data is overfit or underperforming, and a compact, statistically
  filtered feature set is needed to raise cross-validated Q2Y.
when_to_use_negative:
- The analysis goal is univariate association testing (e.g., ANCOVA association analysis
  or IntLIM interaction modeling) rather than PLS-based classification — VIP filtering
  is defined only for projection models
- The feature table is already a small, curated marker panel where additional filtering
  would discard informative variables
- The data lack a phenotype/class response, so supervised PLS-DA and its Q2Y metric
  are not applicable
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: SMART 2.0
  role: Provides the PLS/PLS-DA classification module in which the dual pFDR + top-10
    VIP filtering strategy is applied
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Language in which SMART is implemented and in which the gene/metabolite tables
    are loaded, models are fit, and plsda_results.csv is written
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:ff00ccdf861aae6cb108409d3b6ca27de1226c04c13903573128a97e42fef0b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vip-feature-filtering-for-pls-models

## Summary

A dual-filter variable selection strategy for SMART 2.0 PLS/PLS-DA models: first restrict features to those passing pFDR < 0.05, then keep the top-10 VIP-scored variables, to boost cross-validated predictive accuracy (Q2Y) on integrated omics data.

## When to use

Use when a PLS/PLS-DA classifier on metabolomics or integrated gene-expression + metabolomics data is overfit or underperforming, and a compact, statistically filtered feature set is needed to raise cross-validated Q2Y.

## When NOT to use

- The analysis goal is univariate association testing (e.g., ANCOVA association analysis or IntLIM interaction modeling) rather than PLS-based classification — VIP filtering is defined only for projection models
- The feature table is already a small, curated marker panel where additional filtering would discard informative variables
- The data lack a phenotype/class response, so supervised PLS-DA and its Q2Y metric are not applicable

## Inputs

- Gene-expression matrix (20,254 genes) from the Mathelab/IntLIM GitHub repository breast cancer data
- Metabolomics feature table (504 metabolites) from the Mathelab/IntLIM GitHub repository breast cancer data
- Phenotype/class labels and covariates for the covariate-adjusted PLS/PLS-DA model

## Outputs

- Cross-validated Q2Y values for metabolites-only, integrated, and dual-filter models (reference: 0.454, 0.55, 0.701)
- Fitted-model R2Y (reference: 0.725)
- VIP-ranked feature lists written to plsda_results.csv

## How to apply

Load the gene-expression (20,254 genes) and metabolomics (504 metabolites) tables into R and invoke the SMART 2.0 PLS/PLS-DA classification module. First fit baseline models to establish reference cross-validated Q2Y: metabolites-only (target Q2Y = 0.454) and integrated genes + metabolites (target Q2Y = 0.55). Then apply the dual filter: retain features with pFDR < 0.05, rank survivors by Variable Importance in Projection (VIP), keep the top 10, and refit the covariate-adjusted PLS/PLS-DA model. The rationale is that combining statistical significance (pFDR) with projection-based importance (VIP) prunes noisy high-dimensional features while preserving discriminative ones — this lifted Q2Y to 0.701 with R2Y = 0.725 on the breast cancer data. Write the three Q2Y values plus the VIP-ranked feature lists to plsda_results.csv and verify they match the reported reference values.

## Related tools

- **SMART 2.0** (Provides the PLS/PLS-DA classification module in which the dual pFDR + top-10 VIP filtering strategy is applied) — https://github.com/YuJenL/SMART
- **R** (Language in which SMART is implemented and in which the gene/metabolite tables are loaded, models are fit, and plsda_results.csv is written)

## Evaluation signals

- Cross-validated Q2Y for the metabolites-only PLS-DA model reproduces the reference value 0.454
- Cross-validated Q2Y for the integrated genes + metabolites model reproduces the reference value 0.55
- Dual-filter (pFDR < 0.05 + top-10 VIP) covariate-adjusted model reaches Q2Y = 0.701 and R2Y = 0.725
- VIP-ranked feature lists and the three Q2Y values are written to plsda_results.csv and match the reported reference values
- Filtered feature count is consistent with the pFDR < 0.05 pass set (subset of the 504 metabolites / 20,254 genes) and exactly 10 VIP-selected variables enter the refit model

## Limitations

- The top-10 VIP cutoff is a heuristic validated against reference Q2Y values for this data set; it is not an automatically optimized threshold and may need re-tuning on other data
- Q2Y gains must be confirmed by cross-validation; R2Y/Q2Y on the training data alone cannot rule out overfitting after aggressive feature filtering
- As with any filter cascade, features discarded by pFDR < 0.05 can never re-enter the model, so jointly weak-but-interacting variables may be lost

## Evidence

- [abstract] introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods: "introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
- [readme] Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA).: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] SMART streamlines the complete analysis flow from initial data preprocessing to advanced downstream data analysis: "SMART streamlines the complete analysis flow from initial data preprocessing to advanced downstream data analysis"
