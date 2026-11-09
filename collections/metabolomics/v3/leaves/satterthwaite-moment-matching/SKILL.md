---
name: satterthwaite-moment-matching
description: 'Use when a kernel machine score test needs a null p-value: the quadratic-form
  statistic Q''s null distribution must be approximated by Satterthwaite moment-matching
  as a scaled chi-square c·χ²_ν.'
when_to_use_negative:
- The input is raw metabolite intensities rather than partial residuals from the covariate-adjusted
  null regression — preprocessing (run-day median normalization, kNN imputation with
  k = 10, log transform, standardization) must be done first.
- 'The graph''s connectivity structure has not been examined: the article notes it
  is important to examine graph structure before regularization, and the unregularized
  normalized Laplacian hindered testing performance on disconnected graphs.'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language for the score test and Satterthwaite approximation;
    all simulations were accomplished using R
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:7222b23ac7fdb719a08a1bdc80af073c5ade61475b0dbeff340a304526420659
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# satterthwaite-moment-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pathway-level significance in PaIRKAT by evaluating the quadratic-form kernel score statistic Q and approximating its null distribution with Satterthwaite's moment-matching scaled chi-square (c·χ²_ν) to obtain the p-value for H0: τ=0.

## When to use

Use when a kernel machine score test needs a null p-value: the quadratic-form statistic Q's null distribution must be approximated by Satterthwaite moment-matching as a scaled chi-square c·χ²_ν.

## When NOT to use

- The input is raw metabolite intensities rather than partial residuals from the covariate-adjusted null regression — preprocessing (run-day median normalization, kNN imputation with k = 10, log transform, standardization) must be done first.
- The graph's connectivity structure has not been examined: the article notes it is important to examine graph structure before regularization, and the unregularized normalized Laplacian hindered testing performance on disconnected graphs.

## Inputs

- Per-metabolite partial residuals from linear regression models controlling for white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, and hemoglobin
- Regularized graph kernel matrix (regularized normalized Laplacian) with regularization parameter τ = 1
- KEGG pathway membership/graph definitions (e.g. 28 pathways with measurements on at least 10 metabolites)

## Outputs

- pathway_pvalues.csv containing one p-value entry per tested pathway, each in [0, 1]

## How to apply

Load the per-metabolite partial residuals from the covariate-adjusted null regressions (white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, hemoglobin) and the regularized graph kernel matrix with τ = 1 in R. For each KEGG pathway, compute the kernel-weighted quadratic-form score statistic Q in the residual vector under the null model. Estimate the mean and variance of Q under H0 and apply Satterthwaite moment-matching to fit a scaled chi-square c·χ²_ν, where c and ν are determined from the first two moments of Q. Evaluate the upper-tail probability of the fitted distribution at the observed Q to get each pathway's p-value, then assemble the pathway-level p-values into a table written to pathway_pvalues.csv. Correctness is accepted by reproducibility of the Section 2 score-test computation, since no numeric power or calibration threshold is reported for this step.

## Related tools

- **R** (Implementation language for the score test and Satterthwaite approximation; all simulations were accomplished using R) — https://github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- pathway_pvalues.csv exists and contains exactly one p-value entry per tested pathway
- Every reported p-value falls in the valid [0, 1] range
- Q is computed as the kernel-weighted quadratic form in the residual vector under the null model for each pathway
- The Satterthwaite scaling constants c and ν are determined from the first two moments (mean and variance) of Q under H0
- Reproducibility of the score-test computation as described in Section 2 of the paper (no numeric power or calibration threshold is reported for this single step)

## Limitations

- The regularization parameter τ is fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported.
- Correctness of the Satterthwaite moment-matching is validated only by reproducibility of the described score-test computation; no numeric power or calibration threshold is reported in the paper for this step.
- Some analyzed KEGG pathways had fewer than 10 metabolites because the filtered dataset did not contain every pathway member, and graph structure must be examined before applying regularization.

## Evidence

- [other] Significance of the pathway–phenotype association is computed as a score test, with the null distribution obtained by Satterthwaite's moment-matching approximation to a χ2 distribution: [evidence span withheld — anchor status: not_found]
