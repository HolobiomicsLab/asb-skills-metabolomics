---
name: kernel-score-test-statistic-computation
description: 'Use when testing association between a phenotype (e.g., log FEV1/FVC
  ratio or percent emphysema) and kernel-transformed, graph-regularized KEGG pathway
  data and a pathway-level p-value is required under H0: tau=0.'
when_to_use_negative:
- The pathway graph structure has not been examined first — the article stresses it
  is important to examine graph structure before regularization, and the unregularized
  normalized Laplacian hindered testing performance on disconnected graphs
- The pathway has fewer than 10 measured metabolites (the real-data analysis restricted
  to 28 KEGG pathways with measurements on at least 10 metabolites)
- The response is not a continuous phenotype appropriately modeled by the semi-parametric
  kernel regression framework with Gaussian covariate adjustment
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation language for the kernel score test and Satterthwaite approximation
    in the PaIRKAT scripts and example workflow
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:2b2cc38f30d1a8866fbca820138b8c0ce6dfdcc0cff5267bf1d7c886856eff90
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# kernel-score-test-statistic-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pathway-level significance in PaIRKAT by evaluating a kernel-weighted quadratic-form score statistic Q on covariate-adjusted partial residuals and obtaining p-values via Satterthwaite's moment-matched scaled chi-square null distribution.

## When to use

Use when testing association between a phenotype (e.g., log FEV1/FVC ratio or percent emphysema) and kernel-transformed, graph-regularized KEGG pathway data and a pathway-level p-value is required under H0: tau=0.

## When NOT to use

- The pathway graph structure has not been examined first — the article stresses it is important to examine graph structure before regularization, and the unregularized normalized Laplacian hindered testing performance on disconnected graphs
- The pathway has fewer than 10 measured metabolites (the real-data analysis restricted to 28 KEGG pathways with measurements on at least 10 metabolites)
- The response is not a continuous phenotype appropriately modeled by the semi-parametric kernel regression framework with Gaussian covariate adjustment

## Inputs

- Per-metabolite partial residuals from covariate-adjusted null linear regressions (covariates: white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, hemoglobin)
- Regularized graph kernel matrix per pathway (regularized normalized Laplacian, tau = 1), derived from KEGG pathway graphs whose edges come from the KEGG reaction database
- Phenotype vector under test (e.g., log FEV1/FVC ratio or percent emphysema)

## Outputs

- pathway_pvalues.csv — one p-value per tested pathway from the Satterthwaite chi-square approximated score test

## How to apply

Load the per-metabolite partial residuals from linear regressions fit on each metabolite controlling for white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, and hemoglobin, together with the regularized graph kernel matrix built with regularization parameter tau = 1. For each pathway, compute the quadratic-form score statistic Q as the kernel-weighted quadratic form in the residual vector under the null model. Estimate the mean and variance of Q under H0 and apply Satterthwaite's moment-matching to approximate its null distribution as a scaled chi-square c*chi-squared_nu. Evaluate the upper-tail probability of this fitted scaled chi-square at the observed Q to obtain the pathway p-value. Assemble p-values for the pathways (in the COPDGene application, the 28 KEGG pathways with measurements on at least 10 metabolites) into a table and write it to pathway_pvalues.csv.

## Related tools

- **R** (Implementation language for the kernel score test and Satterthwaite approximation in the PaIRKAT scripts and example workflow) — https://github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- pathway_pvalues.csv exists and contains exactly one p-value entry per tested pathway
- Every p-value falls in the valid [0, 1] range
- The score statistic is computed with regularization parameter tau = 1, matching the article's real-data setting
- The null distribution used is a scaled chi-square c*chi-squared_nu whose parameters (c, nu) derive from the first two moments of Q via Satterthwaite moment matching, as described in Section 2 of the paper
- Results are reproducible against the example workflow and source code hosted at github.com/CharlieCarpenter/PaIRKAT

## Limitations

- The regularization parameter tau is fixed to 1 in all simulations and data analyses, with no sensitivity analysis of tau reported
- For datasets with over 1,000 observations, p-values from the compared methods were indistinguishable; performance differences were demonstrated only on random subsets of 100-500 subjects
- Sparsity in metabolomic pathway graphs can greatly hinder performance, so graph structure must be examined before applying regularization
- No numeric power or calibration threshold is reported for this single step; correctness of the Satterthwaite moment-matching is accepted by reproducibility of the score-test computation
- Some mathematical formulas (e.g., the regularization kernel definition) appear as blank placeholders in the article text, and derivation details are deferred to an external reference

## Evidence

- [discussion] Power improvement from regularized Laplacian: [evidence span withheld — anchor status: not_found]
