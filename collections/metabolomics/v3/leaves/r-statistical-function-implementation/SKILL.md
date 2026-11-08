---
name: r-statistical-function-implementation
description: Use when a paper defines a statistical test rule (e.g., a density-ratio
  FDR surface) that must be turned into a reusable R function and validated against
  reported rejection counts.
when_to_use_negative:
- The analysis goal is generic multiple-testing correction (e.g., Benjamini-Hochberg
  q-values) rather than the article's two-dimensional fdr2d rule
- No permutation-based null statistics are available, since r(z) is fit by contrasting
  observed statistics against np permutations
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language in which the fdr2d density-ratio estimator, penalized log-likelihood
    fit, and plug-in surface function are implemented and applied to metabolite test
    statistics
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:8d71049b3a3331f0adef80bc21e9db5e354aa50006a88e092ece9c581fdec928
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# r-statistical-function-implementation

## Summary

Implement and validate a standalone R statistical function by translating a published mathematical rule — here the two-dimensional local FDR (fdr2d) for metabolite biomarker testing — into executable code and checking it against article-reported results.

## When to use

Use when a paper defines a statistical test rule (e.g., a density-ratio FDR surface) that must be turned into a reusable R function and validated against reported rejection counts.

## When NOT to use

- The analysis goal is generic multiple-testing correction (e.g., Benjamini-Hochberg q-values) rather than the article's two-dimensional fdr2d rule
- No permutation-based null statistics are available, since r(z) is fit by contrasting observed statistics against np permutations

## Inputs

- Observed test statistics per metabolite (e.g., t statistics z1 and log standard error z2) from a metabolomics dataset such as Schisandra
- Permutation-based null distribution of the test statistic (np permutations)

## Outputs

- Standalone R function computing the fdr2d surface fdr2d(z) = pi0*r(z)/(np*(1-r(z)))
- Per-metabolite fdr2d values and rejection counts at a target fdr2d threshold

## How to apply

Translate the mathematical definition into an R function: for fdr2d, define the density-ratio r(z) between observed t-statistics (z1) and the log standard error of the statistic (z2) versus np permutation nulls, and fit it by maximizing a penalized log-likelihood with a Ploner-style smooth estimator, choosing the penalty that stabilizes the tails. Estimate the true-null proportion pi0 from the fitted mixture and plug it into the surface fdr2d(z) = pi0*r(z)/(np*(1-r(z))). Run the function on the article's metabolomics test statistics (e.g., the Schisandra dataset) with permutation-based nulls, count rejections at the target fdr2d threshold, and require that standalone-function rejection counts match the counts reported in the article before accepting the implementation.

## Related tools

- **R** (Language in which the fdr2d density-ratio estimator, penalized log-likelihood fit, and plug-in surface function are implemented and applied to metabolite test statistics) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript fdr2d_test.R --tstats schisandra_t.txt --logse schisandra_logse.txt --perm null_perm.txt --threshold 0.05
```

## Evaluation signals

- Rejection counts produced by the standalone R function on the Schisandra dataset match the counts reported in the MetPC article at the target fdr2d threshold
- The penalized log-likelihood fit of r(z) is stable in the tails (no extreme ratio values near r -> 1 that blow up 1/(1-r))
- Estimated pi0 (true-null proportion) lies in the valid range (0, 1) and is consistent with the mixture fit
- fdr2d(z) values are non-negative for all metabolites and monotone in the density ratio r(z)

## Limitations

- Penalty choice for the penalized log-likelihood is a decision point: a poorly chosen penalty destabilizes the r(z) fit in the tails, distorting fdr2d near r -> 1
- Validation in this skill is tied to reproducing Schisandra rejection counts; the method's behavior on other datasets is not established by the source cards
- Requires permutation-based null statistics; without np permutations the density-ratio model cannot be fit

## Evidence

- [methods] fdr2d definition as ratio of null to joint density: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously"
- [abstract] biomarker discovery pipeline context: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
