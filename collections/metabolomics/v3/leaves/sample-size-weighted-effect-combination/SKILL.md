---
name: sample-size-weighted-effect-combination
description: Use when the same feature pairs have Pearson correlation coefficients
  measured in multiple studies, each with its own sample size, and a single combined
  fixed/random-effects estimate per pair is needed.
when_to_use_negative:
- Effect sizes are measured on different scales (e.g. mixing correlation coefficients
  with fold-changes or P-values) and cannot be averaged as correlations
- The question is simply computing per-sample Pearson correlations within one dataset
  — use MOG's pairwise correlation analysis instead of meta-analysis
- Only a single study is available, so there is nothing to combine across studies
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Performs the sample-size-weighted fixed and random effects meta-analysis of
    correlation coefficients and writes meta_analysis_results.csv
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional external engine; MOG can send data subsets to R for additional analyses
    such as meta-analytic combination
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:c4324d4793a68eefb1b2a3528bd5005f3338c6db042594c7bd45ac4308fc496f
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Sample-size-weighted combination of per-study effect sizes (fixed and random effects meta-analysis)

## Summary

Combines per-study Pearson correlation coefficients for the same feature (gene) pair across multiple studies using weights proportional to each study's sample size, under either a fixed effects or a random effects (e.g. DerSimonian-Laird) model, as implemented in MetaOmGraph.

## When to use

Use when the same feature pairs have Pearson correlation coefficients measured in multiple studies, each with its own sample size, and a single combined fixed/random-effects estimate per pair is needed.

## When NOT to use

- Effect sizes are measured on different scales (e.g. mixing correlation coefficients with fold-changes or P-values) and cannot be averaged as correlations
- The question is simply computing per-sample Pearson correlations within one dataset — use MOG's pairwise correlation analysis instead of meta-analysis
- Only a single study is available, so there is nothing to combine across studies

## Inputs

- Table of per-study Pearson correlation coefficients for each gene pair
- Per-study sample sizes (n_i) for every gene pair and study

## Outputs

- meta_analysis_results.csv with one row per gene pair and separate columns for the fixed effects estimate, the random effects estimate, and the heterogeneity statistic

## How to apply

Load the table of per-study Pearson correlation coefficients together with each study's sample size for every gene pair into MetaOmGraph. For each pair, compute the fixed effects estimate as the sample-size-weighted average of the per-study correlations (weight_i = n_i), so larger studies contribute more trusted estimates. If heterogeneity is present, estimate the between-study variance via DerSimonian-Laird and recompute the weighted average with the resulting random effects weights; choose FEM vs REM based on the user's decision point about between-study variability. Back-transform the combined estimates to the correlation scale and record, per pair, the fixed effects estimate, the random effects estimate, and the heterogeneity statistic, writing one row per gene pair to meta_analysis_results.csv with separate columns for FEM and REM estimates.

## Related tools

- **MetaOmGraph** (Performs the sample-size-weighted fixed and random effects meta-analysis of correlation coefficients and writes meta_analysis_results.csv) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional external engine; MOG can send data subsets to R for additional analyses such as meta-analytic combination)

## Evaluation signals

- The fixed effects estimate for each pair equals the weighted average of per-study correlations with weights exactly proportional to sample size (weight_i = n_i); recompute by hand for a few pairs to verify
- meta_analysis_results.csv has one row per gene pair and contains separate columns for the fixed effects estimate, the random effects estimate, and the heterogeneity statistic
- Random effects weights are smaller for large studies than fixed effects weights, so REM estimates move toward the study-mean relative to FEM when heterogeneity is high
- Combined correlation estimates remain within [-1, 1] after back-transforming to the correlation scale
- Pairs with large heterogeneity statistics show larger gaps between their fixed and random effects estimates than homogeneous pairs

## Limitations

- Random effects estimation depends on the between-study variance estimator (e.g. DerSimonian-Laird); with very few studies this variance is poorly estimated
- The method applies to per-study Pearson correlation coefficients of the same gene pair; it does not combine other effect-size types
- MOG is a standalone Java GUI requiring Java Runtime Environment 8+ and optionally R 3.4+, so the meta-analysis runs interactively rather than as a scriptable command-line step

## Evidence

- [abstract] Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] MOG computes pairwise Pearson correlations: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [other] MOG weighting rationale from source finding: [evidence span withheld — anchor status: not_found]
- [readme] MOG is Java software for large expression datasets: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
