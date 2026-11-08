---
name: covariate-adjusted-prediction-modeling
description: Use when the goal is a supervised classification model (e.g., breast
  cancer vs. control) from metabolomics and/or gene-expression data where covariate
  adjustment and feature filtering must be demonstrated against reference metrics
  like Q2Y = 0.701.
when_to_use_negative:
- The task is unsupervised dimensionality reduction or exploratory visualization rather
  than class prediction — use PCA/other reduction modules instead.
- Input data are raw LC-MS vendor files (.raw, .d, mzXML) that still require SMART's
  data import, peak analysis, and preprocessing modules before any PLS-DA modeling
  is meaningful.
- The dataset lacks a categorical phenotype and covariates, so no covariate-adjusted
  PLS-DA comparison can be constructed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: SMART 2.0
  role: Provides the PLS/PLS-DA classification module used to fit the covariate-adjusted,
    dual-filtered prediction model and compute R2Y/Q2Y
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Base R tool whose integrated metabolomics analysis workflow (preprocessing
    through association analysis) SMART 2.0 extends with classification modules
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which the SMART tool is executed and plsda_results.csv
    is written
- name: IntLIM
  role: Source repository providing the formatted breast cancer gene-expression and
    metabolomics data files used as model inputs
  repo: https://github.com/Mathelab/BreastCancerAmbs_GeneMetabolite_Data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:4d3ffb89d7d26734c5cb7ceea3074603518e5795b09c80c78064071cbd7df7be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - IntLIM
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# covariate-adjusted-prediction-modeling

## Summary

Builds covariate-adjusted PLS/PLS-DA classification models in SMART 2.0, applying a dual-filtering variable selection strategy (pFDR < 0.05 combined with a top-10 VIP cutoff) to maximize cross-validated predictive accuracy (Q2Y) on integrated multiomics data such as the breast cancer gene-expression and metabolomics datasets.

## When to use

Use when the goal is a supervised classification model (e.g., breast cancer vs. control) from metabolomics and/or gene-expression data where covariate adjustment and feature filtering must be demonstrated against reference metrics like Q2Y = 0.701.

## When NOT to use

- The task is unsupervised dimensionality reduction or exploratory visualization rather than class prediction — use PCA/other reduction modules instead.
- Input data are raw LC-MS vendor files (.raw, .d, mzXML) that still require SMART's data import, peak analysis, and preprocessing modules before any PLS-DA modeling is meaningful.
- The dataset lacks a categorical phenotype and covariates, so no covariate-adjusted PLS-DA comparison can be constructed.

## Inputs

- Breast cancer gene-expression data file (20,254 genes) from the Mathelab/IntLIM GitHub repository
- Breast cancer metabolomics data file (504 metabolites) from the Mathelab/IntLIM GitHub repository
- Sample class labels (phenotype) and covariate information for adjustment

## Outputs

- plsda_results.csv containing the three Q2Y values (metabolites-only, integrated, dual-filtered)
- VIP-ranked feature lists for the fitted PLS-DA models
- Reported performance metrics: Q2Y = 0.701 and R2Y = 0.725 for the dual-filter model

## How to apply

Fit a baseline PLS-DA model on metabolites only and record its cross-validated Q2Y (reference: 0.454 on the breast cancer data), then fit an integrated model using both genes (20,254) and metabolites (504) and record Q2Y (reference: 0.55). Apply the dual-filtering strategy: select features with pFDR < 0.05, retain the top 10 by VIP score, and refit the covariate-adjusted PLS/PLS-DA model, targeting Q2Y = 0.701 with R2Y = 0.725. The rationale is that filtering on statistical significance (pFDR) plus model contribution (VIP) removes noise features so the integrated model outperforms metabolites-only prediction. Invoke the SMART 2.0 PLS/PLS-DA module in R, load the gene-expression and metabolomics files from the Mathelab/IntLIM GitHub repository, and compare computed metrics to the reference values as a verification step.

## Related tools

- **SMART 2.0** (Provides the PLS/PLS-DA classification module used to fit the covariate-adjusted, dual-filtered prediction model and compute R2Y/Q2Y) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Base R tool whose integrated metabolomics analysis workflow (preprocessing through association analysis) SMART 2.0 extends with classification modules) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which the SMART tool is executed and plsda_results.csv is written)
- **IntLIM** (Source repository providing the formatted breast cancer gene-expression and metabolomics data files used as model inputs) — https://github.com/Mathelab/BreastCancerAmbs_GeneMetabolite_Data

## Examples

```
Rscript -e 'source("SMART.R"); runPLSDA(genes=expr, metabolites=metab, covariates=cov, filter=list(pfdr=0.05, vip_top=10))'
```

## Evaluation signals

- Metabolites-only PLS-DA Q2Y should approximate the reference value 0.454.
- Integrated (genes + metabolites) PLS-DA Q2Y should approximate the reference value 0.55.
- Dual-filter (pFDR < 0.05 plus top-10 VIP) covariate-adjusted model should reach Q2Y = 0.701 and R2Y = 0.725.
- plsda_results.csv contains exactly three Q2Y values plus VIP-ranked feature lists.
- Dual-filter Q2Y must exceed both baseline Q2Y values, confirming the filtering enhances predictive accuracy.

## Limitations

- Reference metrics are dataset-specific: the 0.454/0.55/0.701 Q2Y targets were established on the breast cancer data and will not transfer to other datasets.
- SMART 2.0's planned AI-assisted methodologies were not yet integrated, so feature interpretation remains manual.
- Further biological experiments are warranted to confirm the significance of novel genes and metabolites that have not yet been documented, so top-ranked VIP features should not be treated as validated biomarkers.

## Evidence

- [abstract] With SMART 2.0, we enhanced SMART 1.0 by introducing new analytical modules in targeted metabolomics analysis: "With SMART 2.0, we enhanced SMART 1.0 by introducing new analytical modules in targeted metabolomics analysis"
- [abstract] advanced dimensionality reduction and classification methods: "data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
- [readme] SMART 2.0 classification module scope: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] Breast cancer data availability from IntLIM repo: [evidence span withheld — anchor status: external_doc]
