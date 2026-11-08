---
name: co-expression-analysis-across-studies
description: Use when the same gene/feature pairs have Pearson correlation coefficients
  computed in several studies, each with its own sample size, and a single combined
  correlation estimate per pair (with heterogeneity) is needed.
when_to_use_negative:
- When only a single study is available — there is no between-study variance to estimate
  and simple pairwise correlation (e.g. MOG's co-expression or differential correlation
  analysis) is the right tool
- When the per-study effect sizes are not Pearson correlation coefficients of the
  same gene pairs across studies — sample-size weighting of correlations assumes the
  same feature pair is measured in each study
- When the goal is exploring correlations within one dataset's samples (e.g. batch-effect
  checks) rather than combining effect sizes across studies
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MetaOmGraph
  role: Java workbench that computes per-study correlations and provides the sample-size-weighted
    meta-analysis of correlation coefficients (fixed or random effects) for feature
    pairs across studies
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: optional external engine for additional analyses; MOG can send data subsets
    to R, e.g. for the DerSimonian–Laird between-study variance estimation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:c7c41b2d5ff03bafab34d555480b76067fa6bb89619089d3b7def75d795f9107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# co-expression-analysis-across-studies

## Summary

Combines per-study Pearson correlation coefficients for the same gene pair across multiple expression studies using MetaOmGraph's meta-analysis, weighting each study by its sample size (weight_i = n_i) under either a fixed effects or a random effects model, so larger studies contribute more trusted estimates.

## When to use

Use when the same gene/feature pairs have Pearson correlation coefficients computed in several studies, each with its own sample size, and a single combined correlation estimate per pair (with heterogeneity) is needed.

## When NOT to use

- When only a single study is available — there is no between-study variance to estimate and simple pairwise correlation (e.g. MOG's co-expression or differential correlation analysis) is the right tool
- When the per-study effect sizes are not Pearson correlation coefficients of the same gene pairs across studies — sample-size weighting of correlations assumes the same feature pair is measured in each study
- When the goal is exploring correlations within one dataset's samples (e.g. batch-effect checks) rather than combining effect sizes across studies

## Inputs

- table of per-study Pearson correlation coefficients for every gene pair
- per-study sample size for each gene pair (n_i values used as fixed effects weights)

## Outputs

- meta_analysis_results.csv with one row per gene pair
- per-pair fixed effects correlation estimate
- per-pair random effects correlation estimate
- per-pair between-study heterogeneity statistic

## How to apply

Load the per-study Pearson correlation table for each gene pair together with each study's sample size n_i. For every pair compute the fixed effects estimate as the sample-size-weighted average sum(n_i * r_i)/sum(n_i), reflecting MOG's design where larger studies are trusted more. For the random effects model, first estimate the between-study variance (e.g. via DerSimonian–Laird) and recompute the weighted average with the resulting random effects weights; this captures genuine between-study heterogeneity rather than assuming one common correlation. Back-transform combined estimates to the correlation scale and record per pair the fixed effects estimate, the random effects estimate, and the heterogeneity statistic. Write one row per gene pair with separate fixed and random effects columns to meta_analysis_results.csv.

## Related tools

- **MetaOmGraph** (Java workbench that computes per-study correlations and provides the sample-size-weighted meta-analysis of correlation coefficients (fixed or random effects) for feature pairs across studies) — https://github.com/urmi-21/MetaOmGraph/
- **R** (optional external engine for additional analyses; MOG can send data subsets to R, e.g. for the DerSimonian–Laird between-study variance estimation)

## Examples

```
Rscript -e 'w <- n/sum(n); r_fe <- sum(w*r); write.csv(meta_results, "meta_analysis_results.csv")'
```

## Evaluation signals

- meta_analysis_results.csv exists with exactly one row per gene pair and separate columns for fixed and random effects estimates
- Fixed effects estimate equals the sample-size-weighted mean: sum(n_i * r_i)/sum(n_i) recomputed manually matches the output for each pair
- Random effects weights differ from raw n_i weights when the DerSimonian–Laird between-study variance is nonzero; estimates converge to the fixed effects result when heterogeneity is ~0
- Combined estimates fall within the range of the input per-study correlations (weighted averages cannot leave the convex hull of the r_i)
- Every reported pair has a heterogeneity statistic accompanying the two combined estimates

## Limitations

- Fixed effects weighting by n_i assumes a single common correlation per gene pair across studies; if true heterogeneity exists, the random effects model must be chosen instead
- Results depend on the quality of the per-study Pearson correlations; studies with outlying or batch-affected samples (e.g. the removed LUAD outlier sample in the case study) can distort combined estimates
- The DerSimonian–Laird variance estimate is one of several options; alternative between-study variance estimators may give different random effects weights

## Evidence

- [methods] MOG's meta-analysis of correlation coefficients computes, for each feature pair, a weighted average of the per-study Pearson correlation coefficients: [evidence span withheld — anchor status: not_found]
- [abstract] Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] we used MOG to compute pairwise Pearson correlations among samples from the same biological condition: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [intro] Researchers can send data subsets to R for additional analyses.: "Researchers can send data subsets to R for additional analyses."
- [readme] MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets.: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
