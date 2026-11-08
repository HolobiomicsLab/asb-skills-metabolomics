---
name: biomarker-rejection-count-validation
description: Use when many metabolite hypotheses are tested simultaneously and FDR
  inflation must be reduced via a two-dimensional local fdr rule based on the t statistic
  (z1) and the log standard error of the test statistic (z2).
when_to_use_negative:
- When only a single or few hypotheses are tested, so simultaneous multiple hypothesis
  testing FDR control is unnecessary
- When permutation-based null statistics are unavailable, since the density-ratio
  model r(z) is fit against np permutation nulls
- For the metabolite identification stage of MetPC, which uses a hierarchical statistical
  model with an EM algorithm for latent variables rather than fdr2d control
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for the penalized log-likelihood fit of r(z) and the
    standalone fdr2d function applied to the Schisandra dataset
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:c06c2361ff68b716b93ff83f67f5b33084fa3d9d3fbf486881982f4c0bd85314
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# biomarker-rejection-count-validation

## Summary

Implements and validates the two-dimensional local false discovery rate (fdr2d) test rule from the MetPC metabolomics pipeline, where biomarker metabolite discovery controls fdr2d under simultaneous multiple hypothesis testing. The skill centers on estimating the null-to-observed density ratio r(z) by penalized log-likelihood and computing the plug-in surface fdr2d(z) = pi0*r(z)/(np*(1-r(z))).

## When to use

Use when many metabolite hypotheses are tested simultaneously and FDR inflation must be reduced via a two-dimensional local fdr rule based on the t statistic (z1) and the log standard error of the test statistic (z2).

## When NOT to use

- When only a single or few hypotheses are tested, so simultaneous multiple hypothesis testing FDR control is unnecessary
- When permutation-based null statistics are unavailable, since the density-ratio model r(z) is fit against np permutation nulls
- For the metabolite identification stage of MetPC, which uses a hierarchical statistical model with an EM algorithm for latent variables rather than fdr2d control

## Inputs

- Vector of observed t statistics (z1) for each metabolite hypothesis
- Vector of log-transformed standard errors of the test statistics (z2)
- Permutation-based null test statistics (np permutations)
- Target fdr2d threshold for rejection counting

## Outputs

- Penalized log-likelihood estimate of the density ratio r(z)
- Estimated proportion of true nulls pi0
- Two-dimensional local fdr surface fdr2d(z) over (z1, z2)
- Count of rejected metabolites at the target fdr2d threshold

## How to apply

Define a two-component density-ratio model r(z) for observed versus permutation-based null test statistics, with z1 the t statistic and z2 the log of the standard error of the test statistic. Fit r(z) in R by maximizing a penalized log-likelihood (Ploner-style smooth estimator), selecting a penalty that stabilizes the fit in the tails. Estimate the proportion of true nulls pi0 from the fitted mixture, generate np permutations of the null statistics, and evaluate the plug-in surface fdr2d(z) = pi0*r(z)/(np*(1-r(z))). Apply the standalone function to the Schisandra dataset test statistics with permutation-based nulls and count rejections at the target fdr2d threshold, mirroring the MetPC biomarker discovery step that controls fdr2d during simultaneous multiple hypothesis testing.

## Related tools

- **R** (Implementation language for the penalized log-likelihood fit of r(z) and the standalone fdr2d function applied to the Schisandra dataset) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- Rejection counts produced by the standalone fdr2d function on the Schisandra dataset must match the rejection counts reported in the article at the target fdr2d threshold
- The fitted ratio r(z) must remain stable in the tails after penalty selection (no divergence of the penalized log-likelihood fit)
- The plug-in surface must satisfy fdr2d(z) = pi0*r(z)/(np*(1-r(z))) with pi0 recoverable from the fitted mixture
- Estimated pi0 must fall in the valid range (0, 1] for the proportion of true nulls
- The permutation nulls used must number np as specified and be exchangeable with the observed (z1, z2) statistics

## Limitations

- Validation depends on exact agreement with Schisandra rejection counts reported in the article; deviations may arise from penalty choice or permutation count np
- The tail behavior of r(z) can be unstable, requiring careful penalty selection in the penalized log-likelihood fit
- The method assumes a two-component (null/alternative) mixture density-ratio model for the test statistics

## Evidence

- [abstract] Biomarker discovery controls the two-dimensional false discovery rate (fdr2d) in simultaneous multiple hypothesis testing.: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [other] fdr2d formulation from the source finding.: [evidence span withheld — anchor status: not_found]
