---
name: fixed-effects-weighted-estimation
description: Use when the same gene (feature) pair has Pearson correlation coefficients
  reported in multiple independent studies with known sample sizes, and a pooled per-pair
  estimate is needed.
when_to_use_negative:
- The correlations come from a single study, so there is nothing to pool across studies
- The per-study effect sizes are not Pearson correlation coefficients (e.g. fold changes
  or p-values), which would require a different effect-size model
- Only sample-level data are available and per-study correlations have not yet been
  computed — compute the pairwise Pearson correlations first (as MOG does) before
  pooling
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MetaOmGraph
  role: Workbench in which the per-study Pearson correlations are computed and the
    sample-size-weighted fixed/random effects meta-analysis of correlation coefficients
    is performed
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional external analysis engine; MOG can send data subsets to R for additional
    analyses such as the meta-analytic combination
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:5e600d251b9ff9135c141206b98f8c04a656561160fcc10902181dd3affcb394
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fixed-effects-weighted-estimation

## Summary

Combines per-study Pearson correlation coefficients for the same feature (gene) pair across multiple studies into a single pooled estimate by weighting each study's correlation proportionally to its sample size, optionally extended to a random effects model with DerSimonian–Laird heterogeneity estimation. This is the meta-analysis mechanism MOG applies when gene pairs are measured across multiple studies.

## When to use

Use when the same gene (feature) pair has Pearson correlation coefficients reported in multiple independent studies with known sample sizes, and a pooled per-pair estimate is needed.

## When NOT to use

- The correlations come from a single study, so there is nothing to pool across studies
- The per-study effect sizes are not Pearson correlation coefficients (e.g. fold changes or p-values), which would require a different effect-size model
- Only sample-level data are available and per-study correlations have not yet been computed — compute the pairwise Pearson correlations first (as MOG does) before pooling

## Inputs

- Table of per-study Pearson correlation coefficients, one value per gene pair per study
- Per-study sample sizes (n_i) aligned with the correlation table
- User's choice of combination model: fixed effects or random effects

## Outputs

- meta_analysis_results.csv with one row per gene pair and separate columns for the fixed effects estimate, random effects estimate, and heterogeneity statistic

## How to apply

Load the table of per-study Pearson correlation coefficients together with each study's sample size n_i for every gene pair. For each pair, compute the fixed effects estimate as the sample-size-weighted average of the per-study correlations, using weight_i = n_i so larger studies contribute more trusted estimates. If requested, estimate the between-study variance via DerSimonian–Laird and recompute the weighted average using the resulting random effects weights. Back-transform the combined estimates to the correlation scale and record, per pair, the fixed effects estimate, the random effects estimate, and a heterogeneity statistic. Write the per-pair results to meta_analysis_results.csv with one row per gene pair and separate columns for the fixed and random effects estimates.

## Related tools

- **MetaOmGraph** (Workbench in which the per-study Pearson correlations are computed and the sample-size-weighted fixed/random effects meta-analysis of correlation coefficients is performed) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional external analysis engine; MOG can send data subsets to R for additional analyses such as the meta-analytic combination)

## Evaluation signals

- Each gene pair in the output has both a fixed effects and a random effects estimate recorded
- The fixed effects estimate for a pair equals sum(n_i * r_i) / sum(n_i) over the input studies for that pair
- Studies with larger n_i pull the pooled estimate closer to their own r_i than small studies do
- The heterogeneity statistic is present for every pair where the random effects model was fit
- Output file is meta_analysis_results.csv with exactly one row per gene pair and separate fixed/random effects columns

## Limitations

- The weighting scheme (weight_i = n_i) assumes study trustworthiness scales with sample size; it does not account for study-level quality differences beyond sample size
- The random effects variant depends on the between-study variance estimate (e.g. DerSimonian–Laird), which is unreliable with very few studies
- Back-transformation to the correlation scale can distort estimates if the combination was performed on a transformed scale

## Evidence

- [other] MOG's meta-analysis of correlation coefficients computes, for each feature pair, a weighted average of the per-study Pearson correlation coefficients, with weights proportional to each study's sample size: [evidence span withheld — anchor status: not_found]
- [abstract] Statistical analyses include co-expression analysis: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] MOG computes pairwise Pearson correlations among samples: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [abstract] MOG sends data subsets to R for additional analyses: "Researchers can send data subsets to R for additional analyses."
- [readme] R is an optional prerequisite of MOG: [evidence span withheld — anchor status: external_doc]
