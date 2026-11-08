---
name: permutation-null-approximation
description: Use when testing many metabolite hypotheses (e.g., biomarker discovery)
  and a standard single-statistic FDR is inflated; input includes observed t statistics
  (z1) with their log standard errors (z2) plus permutation-based null statistics.
when_to_use_negative:
- The analysis goal is metabolite identification rather than biomarker discovery —
  the MetPC pipeline handles identification separately with a hierarchical statistical
  model and EM algorithm
- Only a single hypothesis or a small number of hypotheses are tested, where FDR inflation
  from multiplicity is not a concern
- No permutation-based null statistics are available and they cannot be generated
  for the test statistics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the penalized log-likelihood fit of r(z) (Ploner-style
    smooth estimator) and the plug-in fdr2d computation
- name: MetPC
  role: Biomarker discovery pipeline whose multiple-testing step applies the fdr2d
    control based on the permutation-null approximation
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:c64619c1a5d99c55a2c6c62e7042ea5e99a30f2849ccf45c68617ffb707a6ad0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# permutation-null-approximation

## Summary

Constructs an empirical null distribution of test statistics via permutations and estimates a two-dimensional local false discovery rate (fdr2d) by plugging a penalized-likelihood density ratio r(z) into fdr2d(z) = pi0*r(z)/(np*(1-r(z))), reducing FDR inflation in simultaneous metabolite hypothesis testing.

## When to use

Use when testing many metabolite hypotheses (e.g., biomarker discovery) and a standard single-statistic FDR is inflated; input includes observed t statistics (z1) with their log standard errors (z2) plus permutation-based null statistics.

## When NOT to use

- The analysis goal is metabolite identification rather than biomarker discovery — the MetPC pipeline handles identification separately with a hierarchical statistical model and EM algorithm
- Only a single hypothesis or a small number of hypotheses are tested, where FDR inflation from multiplicity is not a concern
- No permutation-based null statistics are available and they cannot be generated for the test statistics

## Inputs

- Vector of observed t statistics (z1) for each metabolite hypothesis
- Log standard error of each test statistic (z2)
- Permutation-based null test statistics from np permutations
- Target fdr2d threshold

## Outputs

- Estimated density ratio surface r(z) with fitted penalty parameter
- Estimated proportion of true nulls (pi0)
- fdr2d significance surface fdr2d(z) = pi0*r(z)/(np*(1-r(z)))
- Count/list of metabolites rejected at the target fdr2d threshold

## How to apply

Define the two-component density-ratio model r(z) comparing observed test statistics to permutation-based null statistics, where z1 is the t statistic and z2 is the log standard error of the test statistic. Fit r(z) in R by maximizing a penalized log-likelihood (Ploner-style smooth estimator), selecting the penalty so the fit is stabilized in the tails. Estimate the proportion of true nulls pi0 from the fitted mixture and compute the plug-in surface fdr2d(z) = pi0*r(z)/(np*(1-r(z))) using np permutations. Apply fdr2d to the observed (z1, z2) pairs and declare metabolites significant below the chosen fdr2d threshold. The rationale is that conditioning on both the statistic and its standard error controls the two-dimensional false discovery rate when testing multiple hypotheses simultaneously, as done in the MetPC biomarker discovery step.

## Related tools

- **R** (Implementation language for the penalized log-likelihood fit of r(z) (Ploner-style smooth estimator) and the plug-in fdr2d computation)
- **MetPC** (Biomarker discovery pipeline whose multiple-testing step applies the fdr2d control based on the permutation-null approximation) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- Rejection counts produced by the standalone R function on the Schisandra dataset match the rejection counts reported in the article at the target fdr2d threshold
- The fitted r(z) is stable in the tails (the selected penalty smooths tail behavior rather than following noisy permutation counts)
- Estimated pi0 lies in (0, 1] and the fdr2d surface satisfies 0 <= fdr2d(z) <= 1 across the (z1, z2) grid
- fdr2d decreases monotonically toward the extremes of z1 where signal is strongest
- Applied at threshold t, the reported FDR control holds when metabolites are tested simultaneously

## Limitations

- Requires np permutations to build the null distribution; too few permutations destabilize the density-ratio fit in the tails
- The penalized log-likelihood fit depends on penalty selection; a poorly chosen penalty can over- or under-smooth r(z) in the tails
- The plug-in fdr2d assumes the two-component mixture model (null density f0 vs joint density f) adequately describes the (z1, z2) distribution

## Evidence

- [methods] fdr2d ratio definition: [evidence span withheld — anchor status: not_found]
- [abstract] biomarker discovery under fdr2d control: "For biomarker metabolite discovery, our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [methods] penalized likelihood fit: [evidence span withheld — anchor status: not_found]
