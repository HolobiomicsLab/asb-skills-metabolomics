---
name: imputed-omics-data-handling
description: Use when transcriptomics or metabolomics matrices contain missing values
  before MOFA integration in BiomiX, and a decision must be made between imputing
  (NIPALS, random forest, lasso, mean/median, 0) or passing unimputed data to MOFA.
when_to_use_negative:
- Input is already fully imputed — applying a second imputation method would double-correct
  values and bias the variance decomposition
- The downstream integration method cannot handle missing values natively and the
  chosen imputation method introduces unquantified bias into the data
- Single-omics differential analysis (DESeq2/Limma, Wilcoxon) is the goal, where the
  imputation-for-MOFA-integration decision is not the operative step
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiomiX
  role: Provides the imputation Shiny interface (NIPALS, random forest, lasso, mean/median,
    0 values) and feeds imputed or unimputed omics matrices into the MOFA integration
    pipeline
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Consumes imputed or unimputed matrices and can manage missing data natively
    during multi-omics factor analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:d423b38c5a0cef6f6e8eb586a3315baf2d7699dd0c1a2aeeeb06954a97cfba2d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - MOFA (Multi-Omics Factor Analysis)
  - R
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# imputed-omics-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles missing values in omics matrices destined for multi-omics integration by either imputing them (NIPALS, random forest, lasso, mean/median, or zero-filling) via the BiomiX imputation interface, or leaving data unimputed because MOFA can manage missing values natively during integration.

## When to use

Use when transcriptomics or metabolomics matrices contain missing values before MOFA integration in BiomiX, and a decision must be made between imputing (NIPALS, random forest, lasso, mean/median, 0) or passing unimputed data to MOFA.

## When NOT to use

- Input is already fully imputed — applying a second imputation method would double-correct values and bias the variance decomposition
- The downstream integration method cannot handle missing values natively and the chosen imputation method introduces unquantified bias into the data
- Single-omics differential analysis (DESeq2/Limma, Wilcoxon) is the goal, where the imputation-for-MOFA-integration decision is not the operative step

## Inputs

- Transformed transcriptomics matrix (possibly with missing values)
- Transformed metabolomics matrix (possibly with missing values)
- Choice of imputation method: NIPALS, random forest, lasso, mean/median, or 0 values

## Outputs

- Imputed (or deliberately unimputed) omics matrices ready for MOFA integration
- Three-factor MOFA model (mofa_model.hdf5) and per-factor variance/Wilcoxon statistics (mofa_factor_statistics.csv) used for validation

## How to apply

Load the transformed omics matrix into the BiomiX imputation Shiny interface and select one imputation method: NIPALS, random forest, lasso, mean/median, or 0-value filling, while inspecting any bias the chosen method introduces into the data. Alternatively, in BiomiX version 2.4 onward, feed the unimputed matrix directly to MOFA, which can handle missing data during integration. In the PTB case study, the transformed (optionally imputed) transcriptomics and metabolomics matrices were passed to MOFA tuning mode, where the final three-factor model reproduced reported statistics (factor 1: p.adj = 0.0011; 2.23% metabolomics and 33.22% transcriptomics variance explained). The rationale is that the choice between imputation and native missing-value handling changes downstream factor structure, so the same discriminative statistics should be checked after either path. Validation: the fitted model's factor statistics must match the reported variance fractions and adjusted p-value within the values stated in the paper.

## Related tools

- **BiomiX** (Provides the imputation Shiny interface (NIPALS, random forest, lasso, mean/median, 0 values) and feeds imputed or unimputed omics matrices into the MOFA integration pipeline) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Consumes imputed or unimputed matrices and can manage missing data natively during multi-omics factor analysis)

## Evaluation signals

- The exported MOFA model contains exactly three factors, as required by the tuning-mode stopping criterion
- Factor-1 statistics reproduce the reported values: p.adj = 0.0011, ~2.23% metabolomics and ~33.22% transcriptomics variance explained
- The overall three-factor model variance decomposition matches the expected ~5.05% metabolomics / ~45.77% transcriptomics
- Only factor 1 significantly discriminates PTB vs HC under Wilcoxon testing with FDR correction
- If imputation was used, the bias introduced by the chosen method (NIPALS, random forest, lasso, mean/median, or 0) was inspected in the imputation interface before integration

## Limitations

- Users must carefully review the dataset before using BiomiX; there is no automated validation of imputation choices ('We strongly recommend users to follow the guidelines and to carefully review the dataset before using BiomiX.')
- The imputation interface warns about 'any potential bias introduced into the data' by the selected method
- BiomiX does not support instrument normalization for metabolomics, so pre-existing missingness/quality issues must be handled in the Preview-QC interface before imputation decisions

## Evidence
