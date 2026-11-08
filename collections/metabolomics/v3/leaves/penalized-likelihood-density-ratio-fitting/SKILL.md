---
name: penalized-likelihood-density-ratio-fitting
description: 'Use when many metabolite hypotheses are tested simultaneously and raw
  p-value FDR is inflated: you have observed t statistics (z1), log standard errors
  (z2), and np permutations of null statistics, and need a calibrated local fdr2d
  threshold for biomarker discovery.'
when_to_use_negative:
- Input is already a curated biomarker list or a feature table without test statistics
  and permutations — the density-ratio model needs z1, z2, and np permutation nulls.
- The analysis is a single-hypothesis or few-hypothesis test where two-dimensional
  FDR control over simultaneous metabolite tests is unnecessary.
- No permutations can be generated (too few replicates), since r(z) is estimated against
  the permutation-based null density f0.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MetPC
  role: Pipeline whose biomarker discovery step controls fdr2d using the penalized
    density-ratio fit; implemented in R
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:4aa523ef303604133e6af46b13c9fc94b085098e91bd9d8e56170a086807f3c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  - R
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# penalized-likelihood-density-ratio-fitting

## Summary

Fits a smooth two-component density-ratio model r(z) between observed and permutation-based null test statistics via penalized log-likelihood, then plugs the fit into the two-dimensional local false discovery rate fdr2d(z) = pi0*r(z)/(np*(1-r(z))) to control FDR in simultaneous metabolite hypothesis testing.

## When to use

Use when many metabolite hypotheses are tested simultaneously and raw p-value FDR is inflated: you have observed t statistics (z1), log standard errors (z2), and np permutations of null statistics, and need a calibrated local fdr2d threshold for biomarker discovery.

## When NOT to use

- Input is already a curated biomarker list or a feature table without test statistics and permutations — the density-ratio model needs z1, z2, and np permutation nulls.
- The analysis is a single-hypothesis or few-hypothesis test where two-dimensional FDR control over simultaneous metabolite tests is unnecessary.
- No permutations can be generated (too few replicates), since r(z) is estimated against the permutation-based null density f0.

## Inputs

- Vector of observed test statistics (z1: t statistics per metabolite)
- Vector of log standard errors of the test statistics (z2)
- Permutation-based null test statistics (np permutations)
- Target fdr2d threshold
- Metabolomics intensity/feature table from which the statistics are computed

## Outputs

- Fitted penalized density-ratio function r(z1, z2)
- Estimated proportion of true nulls (pi0)
- fdr2d surface / per-metabolite fdr2d values
- List of rejected metabolites (biomarker candidates) at the target fdr2d threshold

## How to apply

Define the two-component density-ratio model r(z), the ratio of the null density f0 to the joint density f evaluated at (z1, z2), where z1 is the t statistic and z2 is the log standard error of the test statistic. Fit r(z) by maximizing a penalized log-likelihood in R (Ploner-style smooth estimator), selecting a penalty large enough to stabilize the fit in the tails where permutation nulls are sparse. Estimate the proportion of true nulls pi0 from the fitted mixture, generate np permutations of the observed statistics as the empirical null, and compute the plug-in surface fdr2d(z) = pi0*r(z)/(np*(1-r(z))). Apply the fitted fdr2d rule to the observed test statistics (e.g., the Schisandra dataset) and count rejections at the target fdr2d threshold. This two-dimensional formulation is used because it reduces FDR inflation relative to univariate local fdr when testing many metabolite hypotheses simultaneously.

## Related tools

- **MetPC** (Pipeline whose biomarker discovery step controls fdr2d using the penalized density-ratio fit; implemented in R) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
fdr2d <- pi0 * r / (np * (1 - r))  # plug-in surface from penalized-fit r(z) on (t statistic, log SE) pairs
```

## Evaluation signals

- Rejection counts produced by the standalone fdr2d function match the Schisandra dataset rejection counts reported in the article at the target fdr2d threshold.
- The fitted r(z) is stable in the tails (penalty chosen so the penalized log-likelihood fit does not oscillate where permutation nulls are sparse).
- Estimated pi0 falls in (0, 1] and the plug-in surface pi0*r(z)/(np*(1-r(z))) is bounded and monotone in r(z).
- Permutation null density is based on np permutations and reproduces the null distribution of (z1, z2) pairs.
- Rejections at a fixed threshold do not exceed the nominal two-dimensional FDR control level.

## Limitations

- Requires np permutations of the test statistics; permutation count and tail sparsity drive the stability of the penalized fit.
- The method is specific to the two-component model with z1 = t statistic and z2 = log standard error; other statistic pairs are not validated in this article.
- The article's discussion content was not extractable, so caveats beyond the abstract and source-card description are not documented.

## Evidence

- [other] fdr2d definition and formula: [evidence span withheld — anchor status: not_found]
- [abstract] abstract fdr2d purpose: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [abstract] abstract biomarker discovery scope: "For biomarker metabolite discovery, our procedure controls two-dimensional false discovery rate (fdr2d)"
