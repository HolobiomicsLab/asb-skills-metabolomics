---
name: kernel-machine-score-testing
description: Use when testing phenotype–pathway (or feature-set) associations for
  metabolomics or similar high-dimensional 'Z' features whose dependence on the outcome
  is modeled as a kernel-weighted random effect rather than fixed covariates.
when_to_use_negative:
- The graph is disconnected and you plan to use the unregularized normalized Laplacian
  kernel — the paper shows this hinders testing performance; use the regularized version
  or ignore topography
- You only have fixed covariate effects with no kernel/random-effect structure — ordinary
  linear regression suffices
- The full cohort exceeds ~1,000 observations and you expect to discriminate between
  kernel variants on a single full-data test — p-values were indistinguishable at
  that size; use downsampled subsets
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the kernel machine score test and all simulations
  repo: github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Builds pathway graphs (Barabasi-Albert random graphs in simulations; KEGG
    reaction-based graphs in COPDGene analysis) used to compute the normalized Laplacian
  repo: github.com/CharlieCarpenter/PaIRKAT
- name: KEGGREST
  role: Fetches KEGG human pathway membership via keggLink to define the metabolite
    sets tested
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:b94c73be480e0810ba801134210648ffccff370b0538be2795f50deed8d81c23
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  - impute (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# kernel-machine-score-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits the semi-parametric kernel machine model g(Y)=Xβ+h(Z)+ϵ and computes a variance-component score test p-value, where h(Z) is a random effect with covariance proportional to a kernel matrix (e.g., a Gaussian kernel built on pathway features smoothed by a regularized normalized Laplacian, as in PaIRKAT with τ=1).

## When to use

Use when testing phenotype–pathway (or feature-set) associations for metabolomics or similar high-dimensional 'Z' features whose dependence on the outcome is modeled as a kernel-weighted random effect rather than fixed covariates.

## When NOT to use

- The graph is disconnected and you plan to use the unregularized normalized Laplacian kernel — the paper shows this hinders testing performance; use the regularized version or ignore topography
- You only have fixed covariate effects with no kernel/random-effect structure — ordinary linear regression suffices
- The full cohort exceeds ~1,000 observations and you expect to discriminate between kernel variants on a single full-data test — p-values were indistinguishable at that size; use downsampled subsets

## Inputs

- Phenotype/response vector Y and covariate matrix X (e.g., log FEV1/FVC or percent emphysema with covariates)
- Feature matrix Z of pathway metabolites (preprocessed: run-day median normalization, kNN imputation k=10, log-transformed and standardized, partial residuals from per-metabolite covariate regressions)
- Kernel matrix K (e.g., Gaussian kernel on regularized-Laplacian-smoothed pathway features, ρ = median pairwise distance, τ = 1)

## Outputs

- Per-pathway score test statistic and p-value (one per pathway, e.g., the 28 KEGG pathways)
- Empirical type I error rates (e.g., type1_error_rates.csv from 1000 Monte Carlo replicates per scenario)
- Empirical power estimates per method × scenario × SNR combination (e.g., simulation_power_results.csv)

## How to apply

First estimate covariate effects β under the null by regressing g(Y) on X, then treat h(Z) as a random effect whose covariance is proportional to the kernel matrix K. Build K from the features of interest — in PaIRKAT this is a Gaussian (radial basis) kernel on pathway metabolite features projected through the regularized normalized Laplacian, with bandwidth ρ set to the median pairwise distance and regularization τ=1. Compute the kernel score test statistic for the variance component and its p-value as described in Section 2 of the paper, evaluating at a chosen alpha level (0.05 in the simulations). For power comparisons against methods that ignore topography or use the unregularized normalized Laplacian, run the same score test under all conditions and compare rejection rates or per-pathway p-values. On real data with >1,000 observations, p-values from competing kernels may be indistinguishable, so average p-values over random subsets (e.g., 100 subsets of sizes 100–500) to expose method differences.

## Related tools

- **R** (Implementation language for the kernel machine score test and all simulations) — github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Builds pathway graphs (Barabasi-Albert random graphs in simulations; KEGG reaction-based graphs in COPDGene analysis) used to compute the normalized Laplacian) — github.com/CharlieCarpenter/PaIRKAT
- **KEGGREST** (Fetches KEGG human pathway membership via keggLink to define the metabolite sets tested)

## Examples

```
Rscript SimulationFunctions/TypeI.R  # run null simulations and write type1_error_rates.csv
```

## Evaluation signals

- Under the null (no phenotype association), empirical type I error across 1000 Monte Carlo simulations per scenario is approximately 0.05 (nominal alpha), matching Tables 1–4 and S1–S4
- For each pathway tested, a score test statistic and p-value are produced (e.g., 28 KEGG pathway rows in the COPDGene results table)
- Power comparison shows PaIRKAT reaching 80% power at SNR ≈ 0.23 versus ≈ 0.60 for ignoring network information, with the unregularized normalized Laplacian failing to reach 80% on disconnected graphs
- Per-pathway p-values tabulated across the three methods (ignore topography, normalized Laplacian, regularized normalized Laplacian τ=1) reproduce the ordering seen in Figures 3, S1, S2 (e.g., PaIRKAT improves power for 18/28 pathways for log FEV1/FVC)

## Limitations

- The regularization parameter τ was fixed at 1 in all simulations and data analyses, with no sensitivity analysis of τ reported
- Mathematical equations (e.g., the regularization kernel definition) appear as blank placeholders in the source text; derivation details are deferred to an external reference
- Real-data performance differences between methods were only demonstrated on downsampled subsets (100–500 subjects); p-values were indistinguishable on the full >1,000-observation datasets
- It is important to examine graph structure before applying regularization — the method is not universally applicable without inspection, and graph sparsity in metabolomic pathways can greatly hinder performance

## Evidence

- [other] Fit the semi-parametric kernel machine model: [evidence span withheld — anchor status: not_found]
- [other] Compute the kernel score test statistic: [evidence span withheld — anchor status: not_found]
