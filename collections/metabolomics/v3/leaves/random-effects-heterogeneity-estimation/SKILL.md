---
name: random-effects-heterogeneity-estimation
description: Use when the same gene pairs have Pearson correlation coefficients measured
  in multiple studies, each with its own sample size, and you need pooled fixed and
  random effects estimates with a heterogeneity statistic.
when_to_use_negative:
- Each gene pair is measured in only a single study — there is nothing to pool and
  no heterogeneity to estimate
- The effect sizes are differential expression fold-changes or other non-correlation
  statistics rather than per-study Pearson correlation coefficients
- The analysis goal is within-study co-expression (e.g. the Spearman >= 0.60 gene-pair
  screen in MOG) rather than combining estimates across studies
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaOmGraph
  role: Interactive workbench in which the sample-size-weighted fixed and random effects
    meta-analysis of correlation coefficients is computed and configured by the user
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional external engine to which MOG data subsets can be sent for additional
    analyses such as the heterogeneity estimation
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:7780bb84d1c6a7e9d7124ea35fda0680c01573603569dc9dc996ff9874388551
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MetaOmGraph
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# random-effects-heterogeneity-estimation

## Summary

Combines per-study Pearson correlation coefficients for the same gene pair via sample-size-weighted meta-analysis, estimating between-study heterogeneity (e.g. DerSimonian-Laird) to yield both fixed effects and random effects combined estimates. In MetaOmGraph (MOG) workflows this is how replicated gene-pair correlation evidence across multiple studies is consolidated.

## When to use

Use when the same gene pairs have Pearson correlation coefficients measured in multiple studies, each with its own sample size, and you need pooled fixed and random effects estimates with a heterogeneity statistic.

## When NOT to use

- Each gene pair is measured in only a single study — there is nothing to pool and no heterogeneity to estimate
- The effect sizes are differential expression fold-changes or other non-correlation statistics rather than per-study Pearson correlation coefficients
- The analysis goal is within-study co-expression (e.g. the Spearman >= 0.60 gene-pair screen in MOG) rather than combining estimates across studies

## Inputs

- Table of per-study Pearson correlation coefficients for each gene pair
- Per-study sample sizes n_i for each gene pair

## Outputs

- meta_analysis_results.csv with one row per gene pair, separate columns for fixed effects estimate, random effects estimate, and heterogeneity statistic

## How to apply

Load the per-study Pearson correlation coefficients together with each study's sample size n_i for every gene pair. For each pair, compute the fixed effects estimate as the sample-size-weighted average (weight_i = n_i), so larger studies contribute more trusted estimates. Estimate the between-study variance (e.g. by the DerSimonian-Laird method) and recompute the weights to obtain the random effects weighted average. Back-transform the combined estimates to the correlation scale and record, per gene pair, the fixed effects estimate, the random effects estimate, and the heterogeneity statistic. In MOG the user may choose between a fixed effects or a random effects model, and results are written to meta_analysis_results.csv with one row per gene pair and separate columns for the fixed and random effects estimates.

## Related tools

- **MetaOmGraph** (Interactive workbench in which the sample-size-weighted fixed and random effects meta-analysis of correlation coefficients is computed and configured by the user) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional external engine to which MOG data subsets can be sent for additional analyses such as the heterogeneity estimation) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
R> fe <- weighted.mean(r_per_study, n_per_study)  # sample-size-weighted fixed effects correlation; REM via DerSimonian-Laird weights, results written to meta_analysis_results.csv
```

## Evaluation signals

- Fixed effects estimate equals sum(n_i * r_i) / sum(n_i) for each gene pair when recomputed independently from the per-study table
- Random effects weights are <= the fixed effects sample-size weights for heterogeneous pairs, converging to them as the estimated between-study variance approaches zero
- meta_analysis_results.csv has exactly one row per gene pair with non-empty fixed effects, random effects, and heterogeneity columns and estimates on the correlation scale (roughly within [-1, 1])
- Heterogeneity statistics are consistent with the gap between fixed and random effects estimates for each pair (large heterogeneity implies a visibly different random effects estimate)
- Pooled estimates from the two models are identical for gene pairs whose per-study correlations are nearly identical across studies

## Limitations

- Requires per-study sample sizes; weighting (weight_i = n_i) is only as trustworthy as the reported study sizes
- Random effects depends on the between-study variance estimator (e.g. DerSimonian-Laird), which is unstable with very few studies per gene pair
- The choice between fixed and random effects is left to the user and is not made automatically by MOG

## Evidence

- [other] Source card finding on MOG correlation meta-analysis: [evidence span withheld — anchor status: not_found]
- [results] MOG as the exclusive analysis engine: "The statistical analyses and visualizations shown were generated exclusively using MOG."
- [abstract] R as optional external analysis engine: "Researchers can send data subsets to R for additional analyses."
- [readme] README prerequisites for running MOG with R: "Java Runtime Environment 8 (or higher)
* R 3.4 (or higher) [optional]"
