---
name: cohort-discriminant-factor-testing
description: Use when a trained MOFA model (e.g., 2-3 factors from BiomiX tuning mode
  on transcriptomics + metabolomics matrices) must be screened for factors that significantly
  separate two cohort conditions, and the per-omics variance captured by those factors
  must be quantified.
when_to_use_negative:
- The goal is single-omics differential analysis (e.g., DESeq2/Limma transcriptomics
  or Wilcoxon metabolomics peak testing) rather than testing latent factors from an
  integrated model
- The model was trained with an integration method without an automatic factor-tuning/discrimination
  workflow (e.g., SNF or NEMO clustering in BiomiX 2.5), which lacks MOFA-style factor
  statistics
- Only one condition/cohort is present — Wilcoxon cohort discrimination is undefined
  without two groups
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: Runs MOFA in tuning mode, performs Wilcoxon-based factor discrimination with
    FDR correction, and exports mofa_model.hdf5 and mofa_factor_statistics.csv
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Latent factor model whose factors are tested for cohort discrimination and
    whose per-view variance explained is quantified
- name: R
  role: Implementation language of the BiomiX pipeline executing the Wilcoxon/FDR
    factor testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:96d270b4c9867eb49d7c922440bc3830d7c15c611c2b63454e414b16eb3c434f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cohort-discriminant-factor-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether MOFA latent factors trained on integrated multi-omics matrices significantly discriminate patient cohorts (e.g., PTB patients vs healthy controls), using Wilcoxon tests with FDR correction and per-omics variance-explained fractions to select and validate the discriminant factor.

## When to use

Use when a trained MOFA model (e.g., 2-3 factors from BiomiX tuning mode on transcriptomics + metabolomics matrices) must be screened for factors that significantly separate two cohort conditions, and the per-omics variance captured by those factors must be quantified.

## When NOT to use

- The goal is single-omics differential analysis (e.g., DESeq2/Limma transcriptomics or Wilcoxon metabolomics peak testing) rather than testing latent factors from an integrated model
- The model was trained with an integration method without an automatic factor-tuning/discrimination workflow (e.g., SNF or NEMO clustering in BiomiX 2.5), which lacks MOFA-style factor statistics
- Only one condition/cohort is present — Wilcoxon cohort discrimination is undefined without two groups

## Inputs

- Fitted MOFA model object from BiomiX tuning mode (trained on transformed transcriptomics and metabolomics matrices, with missing values handled natively or imputed via NIPALS, random forest, lasso, or mean/median)
- Cohort condition metadata (e.g., PTB vs healthy control labels) to test factor discrimination

## Outputs

- Per-factor Wilcoxon discrimination statistics with FDR-adjusted p-values (mofa_factor_statistics.csv)
- Fitted MOFA model exported to mofa_model.hdf5 (exactly the selected number of factors, e.g., three)
- Variance-explained fractions per omics view per factor (e.g., 2.23% metabolomics / 33.22% transcriptomics for factor 1)

## How to apply

After fitting MOFA models in BiomiX tuning mode (iteratively increasing factor counts until at least three models have a last factor explaining <1% of variance), test each factor's association with the cohort condition using a Wilcoxon test on factor values with False Discovery Rate correction. In the BiomiX PTB vs HC case study, only factor 1 of the selected three-factor model was significant (p.adj = 0.0011), capturing 2.23% of metabolomics and 33.22% of transcriptomics total variance within a model explaining ~5.05% (metabolomics) and ~45.77% (transcriptomics) overall. Select the top-three candidate models by discriminative ability, fit the final model, and export the fitted model (mofa_model.hdf5) plus per-factor variance and Wilcoxon statistics (mofa_factor_statistics.csv) for validation against the reported metrics.

## Related tools

- **BiomiX** (Runs MOFA in tuning mode, performs Wilcoxon-based factor discrimination with FDR correction, and exports mofa_model.hdf5 and mofa_factor_statistics.csv) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Latent factor model whose factors are tested for cohort discrimination and whose per-view variance explained is quantified)
- **R** (Implementation language of the BiomiX pipeline executing the Wilcoxon/FDR factor testing)

## Examples

```
bash Biomix_linux_exe.sh
```

## Evaluation signals

- The exported mofa_model.hdf5 contains exactly the expected number of factors (three in the PTB case study)
- mofa_factor_statistics.csv reproduces the reported factor-1 adjusted p-value (p.adj = 0.0011) and variance fractions (2.23% metabolomics / 33.22% transcriptomics) within the values stated in the paper
- The stopping criterion for tuning was met: at least three models whose last factor explains less than 1% of variance
- Only factors meeting the FDR-corrected significance threshold are flagged as cohort-discriminant (expected: factor 1 only in the PTB case)
- Overall model variance explained is in the expected range (~5.05% metabolomics / ~45.77% transcriptomics)

## Limitations

- Only one factor may significantly discriminate cohorts even in a well-fitting model — non-significant factors should not be over-interpreted
- BiomiX relies on users carefully reviewing datasets before use; no automated validation of inputs is performed
- Interpretation of discriminant factors (clinical correlation, pathway analysis, PubMed mining) is a separate downstream step, not part of the discrimination test itself

## Evidence
