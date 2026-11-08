---
name: pearson-correlation-meta-analysis
description: Use when the same feature/gene pairs have Pearson correlation coefficients
  measured across multiple independent studies (each with its own sample size) and
  they must be combined into a single estimate with fixed or random effects weighting.
when_to_use_negative:
- Each gene pair is measured in only a single study — there is nothing to combine
  and the meta-analysis weights are undefined
- The per-study effect sizes are not Pearson correlation coefficients (e.g. they are
  fold changes or mutual information values); combining them with correlation-specific
  weights is invalid
- The input is raw expression data rather than already-computed per-study correlation
  coefficients and sample sizes — first run MOG's pairwise Pearson correlation analysis
  to produce per-study estimates
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaOmGraph
  role: Computes the pairwise Pearson correlations among samples/conditions and hosts
    the meta-analysis of correlation coefficients as one of its statistical analyses;
    the paper states the analyses were generated exclusively using MOG.
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional backend for additional analyses; data subsets from MOG can be sent
    to R to implement the sample-size-weighted (n_i weights) fixed effects average
    and DerSimonian-Laird random effects combination.
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:9ba88118a784bf438819fed687ccfb452f47333024efe8563395ba562746d2b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pearson-correlation-meta-analysis

## Summary

Combines per-study Pearson correlation coefficients for the same gene pairs into sample-size-weighted fixed and random effects estimates (with between-study heterogeneity) using MetaOmGraph (MOG) and R. This lets large, well-powered studies dominate the combined correlation estimate rather than treating every study equally.

## When to use

Use when the same feature/gene pairs have Pearson correlation coefficients measured across multiple independent studies (each with its own sample size) and they must be combined into a single estimate with fixed or random effects weighting.

## When NOT to use

- Each gene pair is measured in only a single study — there is nothing to combine and the meta-analysis weights are undefined
- The per-study effect sizes are not Pearson correlation coefficients (e.g. they are fold changes or mutual information values); combining them with correlation-specific weights is invalid
- The input is raw expression data rather than already-computed per-study correlation coefficients and sample sizes — first run MOG's pairwise Pearson correlation analysis to produce per-study estimates

## Inputs

- Table of per-study Pearson correlation coefficients, one value per gene pair per study
- Per-study sample sizes (n_i) for each study
- Optional MOG expression project with sample metadata (e.g. Hu-cancer RNA-Seq MOG project with organ and tumor-status metadata) for upstream QC correlations

## Outputs

- meta_analysis_results.csv with one row per gene pair and separate columns for fixed effects estimate, random effects estimate, and heterogeneity statistic
- Sample-sample Pearson correlation matrix (correlation_matrix.csv) when used for sample QC
- Outlier sample report (luad_outlier_report.csv) flagging samples below the >0.70 within-condition correlation threshold

## How to apply

Load the table of per-study Pearson correlation coefficients together with each study's sample size (n_i) for every gene pair. For each pair, compute the fixed effects estimate as the sample-size-weighted average (weight_i = n_i), so larger studies contribute more trusted estimates. Then estimate between-study variance (e.g. DerSimonian-Laird) and recompute the random effects weighted average using the resulting random effects weights. Back-transform combined estimates to the correlation scale and record, per gene pair, the fixed effects estimate, the random effects estimate, and a heterogeneity statistic, writing one row per pair to meta_analysis_results.csv. The same Pearson correlation machinery is also used in MOG for QC — e.g. screening within-condition sample-sample correlations against a >0.70 acceptance threshold, which is how the single LUAD outlier sample was identified and removed from the Hu-cancer-RNASeq-dataset.

## Related tools

- **MetaOmGraph** (Computes the pairwise Pearson correlations among samples/conditions and hosts the meta-analysis of correlation coefficients as one of its statistical analyses; the paper states the analyses were generated exclusively using MOG.) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional backend for additional analyses; data subsets from MOG can be sent to R to implement the sample-size-weighted (n_i weights) fixed effects average and DerSimonian-Laird random effects combination.)

## Examples

```
Rscript -e 'r <- c(0.6,0.65,0.7); n <- c(30,50,120); fe <- sum(n*r)/sum(n); cat(fe, "\n")'  # fixed effects: sample-size-weighted average of per-study Pearson r, then write per-pair results to meta_analysis_results.csv
```

## Evaluation signals

- meta_analysis_results.csv exists with exactly one row per gene pair and separate columns for the fixed effects estimate, random effects estimate, and heterogeneity statistic
- For a given gene pair, the fixed effects estimate equals sum(n_i * r_i) / sum(n_i) — i.e. the sample-size-weighted average with weights proportional to each study's sample size
- A heterogeneity statistic is present for every gene pair, and the random effects estimate differs from the fixed effects estimate for pairs with non-zero between-study variance
- All combined estimates are back-transformed and fall within the correlation scale [-1, 1]
- For the QC use case, all remaining within-condition pairwise Pearson correlations exceed 0.70 and exactly one LUAD sample is flagged as the outlier in luad_outlier_report.csv

## Limitations

- The random effects model requires estimating between-study variance (e.g. DerSimonian-Laird), which is unstable when only a few studies measure the same pair
- Weighting by sample size assumes larger studies give more trusted estimates; if per-study correlation estimates have similar variance regardless of n_i, the weighting is misspecified
- Correlation-based QC against the >0.70 threshold can only flag samples with abnormally low expression relative to condition-mates, as it did for the single LUAD sample; it is not a general batch-effect detector

## Evidence

- [abstract] Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] Within-condition Pearson correlations in the Hu-cancer dataset exceeded 0.70 except one LUAD outlier: [evidence span withheld — anchor status: not_found]
- [results] MOG computed pairwise Pearson correlations among samples from the same biological condition: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [readme] MOG combines large-data analysis with statistical and graphing capabilities: [evidence span withheld — anchor status: external_doc]
