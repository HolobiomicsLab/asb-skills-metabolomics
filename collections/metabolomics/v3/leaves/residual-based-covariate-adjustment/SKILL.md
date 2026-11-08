---
name: residual-based-covariate-adjustment
description: Use when a pathway-level kernel association test (PaIRKAT) must be run
  on metabolomics data that are confounded by clinical covariates, so covariate effects
  must be removed from each metabolite before computing kernel quadratic-form statistics.
when_to_use_negative:
- When covariates are themselves the target of inference — partial residuals remove
  covariate variation, so their effects cannot be tested downstream.
- When the association method already adjusts for covariates internally in a single
  joint regression rather than via a two-stage residual approach.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation language for fitting the per-metabolite covariate-adjusted
    linear regressions and extracting partial residuals
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:c821fe1281a0c8190b46e6f6d6722f7a8c00e1e5b38e41f43a9c319069a16ec8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  repo_url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# residual-based-covariate-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adjusts metabolomic feature intensities for clinical confounders by fitting one linear regression per metabolite on the covariate set and carrying the partial residuals forward as the 'observed' metabolomics data for PaIRKAT's kernel score test.

## When to use

Use when a pathway-level kernel association test (PaIRKAT) must be run on metabolomics data that are confounded by clinical covariates, so covariate effects must be removed from each metabolite before computing kernel quadratic-form statistics.

## When NOT to use

- When covariates are themselves the target of inference — partial residuals remove covariate variation, so their effects cannot be tested downstream.
- When the association method already adjusts for covariates internally in a single joint regression rather than via a two-stage residual approach.

## Inputs

- Standardized metabolite intensity matrix (natural-log transformed, kNN-imputed k=10, run-day median normalized)
- Covariate table with white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, and hemoglobin per subject

## Outputs

- Per-metabolite partial residual matrix used as the observed metabolomics data for the kernel score test

## How to apply

In R, fit a separate linear model for each metabolite with the covariate formula used in the COPDGene analysis: white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, and hemoglobin. Extract the partial residuals from each fit and treat this residual matrix as the observed metabolomics data, replacing the raw (run-day median normalized, kNN-imputed with k=10, natural-log transformed, and standardized) intensities. Downstream, these residuals feed the regularized graph kernel (τ = 1) so the quadratic-form score statistic Q is computed in residual space, making the Satterthwaite χ2-approximated p-value specific to pathway–phenotype association rather than covariate effects. Restrict the fits to complete cases on all covariates — in the paper this left 1,113 FEV1/FVC and 1,065 percent emphysema samples.

## Related tools

- **R** (Implementation language for fitting the per-metabolite covariate-adjusted linear regressions and extracting partial residuals) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
partial_resid <- apply(metab, 2, function(y) residuals(lm(y ~ WBC + pct_eos + pct_lymph + pct_mono + pct_neut + hemoglobin, data = covariates)))
```

## Evaluation signals

- One fitted linear model per metabolite, each sharing the identical covariate formula (six covariates plus intercept).
- Residual vectors are mean-zero (up to numerics) with length equal to the number of complete cases (1,113 for FEV1/FVC; 1,065 for percent emphysema).
- The residual matrix, not raw intensities, is the input to the kernel quadratic-form score statistic Q with τ = 1.
- Downstream pathway_pvalues.csv contains one p-value per tested pathway and every p-value lies in [0, 1].
- Reproducibility check: results match the example workflow published with the PaIRKAT source code.

## Limitations

- The covariate set is phenotype/study-specific (blood-cell counts and hemoglobin for COPDGene); other studies must define their own confounder set.
- Requires complete cases on all covariates — incomplete cases are removed before fitting (n reduced to 1,113 / 1,065 in the paper).
- Software/tool versions for R are not reported (only citation numbers), and no numeric power or calibration threshold is given for this step; correctness is accepted by reproducibility of the described computation.

## Evidence
