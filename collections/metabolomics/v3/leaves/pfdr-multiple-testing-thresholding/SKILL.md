---
name: pfdr-multiple-testing-thresholding
description: Use when fitting PLS/PLS-DA or another supervised classifier on a high-dimensional
  omics table (e.g., 20,254 genes plus 504 metabolites) and you need a defensible,
  two-stage significance-plus-influence cutoff to shrink the feature space before
  refitting.
when_to_use_negative:
- When the goal is per-feature hypothesis testing or p-value combination (e.g., multi-study
  meta-analysis via Pbine) rather than building a predictive classifier
- When the feature table is already small or pre-filtered, where a pFDR-plus-top-10-VIP
  cutoff would discard informative variables unnecessarily
- When the task is untargeted peak detection, annotation, or concentration calibration
  — these belong to SMART 2.0's other modules, not the classification module
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: SMART 2.0
  role: Provides the PLS/PLS-DA classification module in R in which the pFDR < 0.05
    plus top-10-VIP dual filter is applied and Q2Y/R2Y are computed
  repo: https://github.com/YuJenL/SMART
- name: IntLIM
  role: Source repository of the breast cancer gene-expression and metabolomics data
    files used as input to the dual-filter PLS/PLS-DA workflow
  repo: https://github.com/Mathelab/IntLIM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:2f167f93a7910f28f4a81b707c04d4e2d5d63dc66cd371363f75f0f08f7d338b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - IntLIM
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pfdr-multiple-testing-thresholding

## Summary

A two-stage feature-reduction strategy for supervised omics modeling in SMART 2.0: first threshold features by pFDR < 0.05, then retain only the top 10 VIP-ranked variables before refitting a covariate-adjusted PLS/PLS-DA model, which lifted cross-validated Q2Y from 0.55 to 0.701 on the breast cancer gene-expression and metabolomics data.

## When to use

Use when fitting PLS/PLS-DA or another supervised classifier on a high-dimensional omics table (e.g., 20,254 genes plus 504 metabolites) and you need a defensible, two-stage significance-plus-influence cutoff to shrink the feature space before refitting.

## When NOT to use

- When the goal is per-feature hypothesis testing or p-value combination (e.g., multi-study meta-analysis via Pbine) rather than building a predictive classifier
- When the feature table is already small or pre-filtered, where a pFDR-plus-top-10-VIP cutoff would discard informative variables unnecessarily
- When the task is untargeted peak detection, annotation, or concentration calibration — these belong to SMART 2.0's other modules, not the classification module

## Inputs

- Gene-expression feature table (breast cancer example: 20,254 genes)
- Metabolomics feature table (breast cancer example: 504 metabolites)
- Phenotype class labels and covariate definitions for the PLS-DA model

## Outputs

- pFDR-thresholded, VIP-ranked feature list (top 10 VIP variables)
- Refit PLS/PLS-DA performance metrics (cross-validated Q2Y and R2Y)
- Results table (plsda_results.csv) containing the Q2Y values for metabolites-only, integrated, and dual-filter models plus VIP-ranked feature lists

## How to apply

In R, load the breast cancer gene-expression and metabolomics tables and invoke the SMART 2.0 PLS/PLS-DA module. Fit baseline models first to quantify the benefit of filtering: metabolites alone should yield Q2Y ≈ 0.454, and the integrated genes-plus-metabolites model Q2Y ≈ 0.55. Then apply the dual filter: keep only features passing the pFDR < 0.05 threshold, and among those retain the top 10 by VIP (variable importance in projection) score. Refit the covariate-adjusted PLS/PLS-DA model on this reduced set and recompute cross-validated Q2Y (target 0.701) and R2Y (target 0.725). The rationale is that pFDR thresholds control false positives among thousands of candidates while the VIP cutoff keeps variables that actually drive the latent components, rather than filtering on significance alone.

## Related tools

- **SMART 2.0** (Provides the PLS/PLS-DA classification module in R in which the pFDR < 0.05 plus top-10-VIP dual filter is applied and Q2Y/R2Y are computed) — https://github.com/YuJenL/SMART
- **IntLIM** (Source repository of the breast cancer gene-expression and metabolomics data files used as input to the dual-filter PLS/PLS-DA workflow) — https://github.com/Mathelab/IntLIM

## Evaluation signals

- Metabolites-only PLS-DA model must reproduce cross-validated Q2Y ≈ 0.454
- Integrated genes-plus-metabolites model must reproduce Q2Y ≈ 0.55
- Dual-filter (pFDR < 0.05 + top-10 VIP) refit model must reproduce Q2Y ≈ 0.701 with R2Y ≈ 0.725
- plsda_results.csv must contain all three Q2Y values plus VIP-ranked feature lists, and computed metrics must match the reported reference values

## Limitations

- Filtered features are statistical, not validated biology: 'Further biological experiments are warranted to confirm the significance of other novel genes and metabolites that have not yet been documented'
- Demonstrated only on the paper's two example datasets (narcotics and breast cancer); thresholds like the top-10 VIP cutoff may not transfer to other feature dimensionalities
- Q2Y/R2Y gains depend on covariate adjustment and cross-validation design, so omitting the covariate-adjusted refit will not reproduce the reported 0.701

## Evidence

- [abstract] SMART 2.0 classification module context: "introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
- [abstract] Breast cancer data used as demonstrative example: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
- [readme] SMART's Statistical Analysis module includes PLS/PLS-DA classification: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
