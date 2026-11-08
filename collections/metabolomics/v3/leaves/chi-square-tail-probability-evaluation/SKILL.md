---
name: chi-square-tail-probability-evaluation
description: Use when a kernel machine score statistic's null distribution is approximated
  by moment matching and the final deliverable is an upper-tail chi-square p-value
  per pathway.
when_to_use_negative:
- When the input is an effect-size or coefficient estimate rather than a quadratic-form
  score statistic under the null hypothesis
- When the null distribution of the test statistic is known exactly (e.g. from permutation)
  and moment matching to a scaled chi-square would add unnecessary approximation error
- When the goal is to compare p-values across methods on a full cohort of over 1,000
  observations, where the paper reports that P-values from the compared methods were
  indistinguishable; the paper only demonstrated differences via random subsets of
  100–500 subjects
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Language in which the score test with Satterthwaite χ² approximation and upper-tail
    p-value evaluation are implemented
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used to generate the pathway graphs (Barabasi-Albert model) whose regularized
    kernels feed the score statistic for which tail probabilities are computed
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:e8330f4139f7bb5a7ce8abcae79dee7cff0a2dcf5ea29bed931dbcb4b0e70aad
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
  - igraph
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# chi-square-tail-probability-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates the upper-tail probability of a scaled chi-square distribution fitted by Satterthwaite moment matching to a quadratic-form score statistic Q, converting pathway-level kernel association statistics into null-hypothesis p-values in PaIRKAT.

## When to use

Use when a kernel machine score statistic's null distribution is approximated by moment matching and the final deliverable is an upper-tail chi-square p-value per pathway.

## When NOT to use

- When the input is an effect-size or coefficient estimate rather than a quadratic-form score statistic under the null hypothesis
- When the null distribution of the test statistic is known exactly (e.g. from permutation) and moment matching to a scaled chi-square would add unnecessary approximation error
- When the goal is to compare p-values across methods on a full cohort of over 1,000 observations, where the paper reports that P-values from the compared methods were indistinguishable; the paper only demonstrated differences via random subsets of 100–500 subjects

## Inputs

- Quadratic-form score statistic Q per pathway (kernel-weighted quadratic form in the residual vector from per-metabolite covariate-adjusted null regressions)
- Regularized graph kernel matrix for each KEGG pathway, computed with regularization parameter τ = 1
- Per-metabolite partial residuals from linear regression models controlling for white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, and hemoglobin

## Outputs

- Pathway-level p-values in [0, 1], one per tested pathway, assembled into a table (e.g. pathway_pvalues.csv)

## How to apply

After computing the quadratic-form score statistic Q for each pathway's kernel-weighted residuals under the null (with the regularized graph kernel built with τ = 1), estimate the first two moments of Q under H0: τ = 0. Apply Satterthwaite's moment-matching method to approximate the null distribution of Q as a scaled chi-square c·χ²_ν, where c and ν are derived from the mean and variance of Q. The p-value is the upper-tail probability of this fitted scaled chi-square evaluated at the observed Q. This is the significance step of PaIRKAT's semi-parametric kernel machine framework, implemented in R for pathways derived from KEGG human pathways with measurements on at least 10 metabolites.

## Related tools

- **R** (Language in which the score test with Satterthwaite χ² approximation and upper-tail p-value evaluation are implemented) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used to generate the pathway graphs (Barabasi-Albert model) whose regularized kernels feed the score statistic for which tail probabilities are computed) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
Rscript -e 'source("PaIRKAT.R"); pairkat(Z, Y, X, K, tau=1)'
```

## Evaluation signals

- pathway_pvalues.csv exists and contains one p-value entry per tested pathway
- Every reported p-value falls in the valid [0, 1] range
- Number of p-value rows matches the number of KEGG pathways with measurements on at least 10 metabolites (28 in the COPDGene application)
- The fitted scaled chi-square parameters (c, ν) are consistent with the first two moments of the observed Q under the null, so moment matching is reproducible as described in Section 2 of the paper
- P-values are computed with the regularization parameter τ = 1 as specified for the score test

## Limitations

- The regularization parameter τ is fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported in the paper
- Correctness of the Satterthwaite moment matching is accepted by reproducibility of the score-test computation as described in Section 2 of the paper; no numeric power or calibration threshold for this step is reported in the paper
- The scaled chi-square approximation depends on moment matching of the quadratic form Q under H0, so performance depends on whether the graph's sparsity or disconnection was handled by the regularized normalized Laplacian before this step
- Mathematical equations and formulas appear as blank placeholders in the document text (e.g., the regularization kernel definition), so exact formulas for the moments of Q are not available from this source

## Evidence

- [discussion] Significance of the pathway–phenotype association is computed as a score test, with the null distribution obtained by Satterthwaite's moment-matching approximation to a χ2 distribution, yielding the p-value for the test.: [evidence span withheld — anchor status: not_found]
- [discussion] P-values were calculated from a score test as described Section 2 with τ = 1.: "P-values were calculated from a score test as described Section 2 with τ = 1."
- [discussion] The R code for simulations and an example workflow with source code is available at https://github.com/CharlieCarpenter/PaIRKAT: "The R code for simulations and an example workflow with source code is available at https://github.com/CharlieCarpenter/PaIRKAT"
