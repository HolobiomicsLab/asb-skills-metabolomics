---
name: pathway-association-pvalue-calculation
description: Use when per-metabolite partial residuals from null covariate regressions
  and a regularized graph kernel matrix (τ = 1) are available and pathway-level association
  p-values against a phenotype are required.
when_to_use_negative:
- When the pathway graph is disconnected and no regularization step has been applied
  — the unregularized normalized Laplacian hindered testing performance on disconnected
  graphs
- When the goal is omnibus multi-pathway or genome-wide association rather than testing
  individual pathway–phenotype associations within PaIRKAT's semi-parametric kernel
  framework
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: R
  role: Language in which the score test with Satterthwaite chi-square approximation
    is implemented and run
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:40e3969973a723a2f139a14c4463d0b1f43009743681c1c02de2cd7c60222c33
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathway-association-pvalue-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pathway-level p-values for phenotype association in PaIRKAT by forming a kernel-weighted quadratic-form score statistic from covariate-adjusted partial residuals and the regularized graph kernel, then approximating its null distribution with Satterthwaite's moment-matched scaled chi-square.

## When to use

Use when per-metabolite partial residuals from null covariate regressions and a regularized graph kernel matrix (τ = 1) are available and pathway-level association p-values against a phenotype are required.

## When NOT to use

- When the pathway graph is disconnected and no regularization step has been applied — the unregularized normalized Laplacian hindered testing performance on disconnected graphs
- When the goal is omnibus multi-pathway or genome-wide association rather than testing individual pathway–phenotype associations within PaIRKAT's semi-parametric kernel framework

## Inputs

- Per-metabolite partial residual vectors from covariate-adjusted null linear regressions (white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, hemoglobin)
- Regularized graph kernel matrix derived from the pathway's regularized normalized Laplacian with τ = 1
- KEGG pathway membership / graph structure for the 28 pathways with at least 10 measured metabolites

## Outputs

- pathway_pvalues.csv — one p-value entry per tested pathway, each p-value in [0, 1]

## How to apply

In R, take the per-metabolite partial residuals from linear regressions fit on covariates (white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, hemoglobin) together with the regularized graph kernel matrix computed with τ = 1. For each pathway, compute the quadratic-form score statistic Q — the kernel-weighted quadratic form in the residual vector under the null hypothesis H0: τ=0. Estimate the mean and variance of Q under the null and apply Satterthwaite's moment-matching to approximate the null distribution as a scaled chi-square c·χ²_ν, with the scale c and degrees of freedom ν derived from the first two moments of Q. The p-value is the upper-tail probability of that fitted scaled chi-square evaluated at the observed Q, and per the real-data analysis p-values were calculated from this score test 'as described Section 2' with τ = 1.

## Related tools

- **R** (Language in which the score test with Satterthwaite chi-square approximation is implemented and run) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
Rscript run_pairkat_score_test.R --residuals partial_residuals.csv --kernel regularized_kernel_tau1.rds --out pathway_pvalues.csv
```

## Evaluation signals

- pathway_pvalues.csv exists and contains one p-value entry per tested pathway
- Every p-value in pathway_pvalues.csv falls in the valid [0, 1] range
- Each pathway's Q statistic is computed as a kernel-weighted quadratic form in the residual vector under the null model
- The null distribution is a scaled chi-square c·χ²_ν whose c and ν derive from the first two moments of Q (Satterthwaite moment matching)
- Correctness of the Satterthwaite moment-matching is accepted by reproducibility of the score-test computation as described in Section 2 of the paper

## Limitations

- The regularization parameter τ is fixed to 1 for all simulations and analyses, with no sensitivity analysis of τ reported
- No numeric power or calibration threshold is reported for this single step; correctness rests on reproducing the score-test computation
- Graph structure must be examined before applying regularization, so the test is not universally applicable without inspection of sparsity/disconnectivity

## Evidence

- [methods] Significance of the pathway–phenotype association is computed as a score test, with the null distribution obtained by Satterthwaite's moment-matching approximation to a χ2 distribution: [evidence span withheld — anchor status: not_found]
