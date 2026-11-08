---
name: reaction-activity-score-statistics
description: Use when a concordance table of Cohen's kappa and Pearson correlation
  scores (RPS vs RAS, RPS vs FFD) exists and reactions with statistically significant
  RPS–FFD concordance must be identified after multiple-comparison correction.
when_to_use_negative:
- The goal is to compute the raw concordance scores themselves rather than test their
  significance — use the upstream concordanceAnalysis step instead
- Only a small number of reactions are tested and no multiple-comparison correction
  is needed, since the Benjamini-Hochberg step at 5% FDR is specific to family-wise
  multiplicity
- No replicated pairwise-variation distribution is available to bootstrap an empirical
  null, so empirical p-values cannot be formed
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: INTEGRATE
  role: Provides the pipeline context and concordanceAnalysis output that this statistical
    testing skill consumes
  repo: github.com/QLSLab/integrate
- name: Python
  role: Implementation language for the bootstrap resampling, empirical p-value computation,
    and Benjamini-Hochberg adjustment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:a74f416eb8414608038848ca63ed99a20ca1934097ed74fcecc1ad4c2356e74e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# reaction-activity-score-statistics

## Summary

A downstream statistical procedure of the INTEGRATE pipeline that assigns empirical p-values to Cohen's kappa concordance scores (RPS vs RAS / RPS vs FFD) via 1000-fold bootstrap resampling, then applies Benjamini-Hochberg correction at 5% FDR to call significantly concordant reactions.

## When to use

Use when a concordance table of Cohen's kappa and Pearson correlation scores (RPS vs RAS, RPS vs FFD) exists and reactions with statistically significant RPS–FFD concordance must be identified after multiple-comparison correction.

## When NOT to use

- The goal is to compute the raw concordance scores themselves rather than test their significance — use the upstream concordanceAnalysis step instead
- Only a small number of reactions are tested and no multiple-comparison correction is needed, since the Benjamini-Hochberg step at 5% FDR is specific to family-wise multiplicity
- No replicated pairwise-variation distribution is available to bootstrap an empirical null, so empirical p-values cannot be formed

## Inputs

- Concordance table (Cohen's kappa and Pearson correlation scores of RPS vs RAS and RPS vs FFD) from the concordanceAnalysis step
- RPS pairwise-variation distributions per reaction (for bootstrap resampling)

## Outputs

- Table of empirical p-values and Benjamini-Hochberg adjusted p-values per reaction (saved as adjusted_pvalues.csv)
- Filtered list of reactions whose adjusted p-value is below the 5% FDR threshold

## How to apply

Load the concordance table produced by the upstream concordanceAnalysis step, which computes Cohen's kappa (with 'linear' weighting) and Pearson correlation for RPS vs RAS and RPS vs FFD. For each reaction, bootstrap-resample the RPS pairwise-variation distribution 1000 times with replacement to build an empirical null distribution of the kappa concordance score. Assign each reaction an empirical p-value equal to the fraction of bootstrap null kappa values at least as extreme as the observed score. Apply the Benjamini-Hochberg procedure to the empirical p-values at a 5% false discovery rate, then filter to reactions whose adjusted p-value falls below the 5% FDR threshold. In the INTEGRATE proof-of-principle (ENGRO2 model, breast cell lines), this recovered ACONT, ASPTA (forward), RPI, and RPE as the significant reactions.

## Related tools

- **INTEGRATE** (Provides the pipeline context and concordanceAnalysis output that this statistical testing skill consumes) — github.com/QLSLab/integrate
- **Python** (Implementation language for the bootstrap resampling, empirical p-value computation, and Benjamini-Hochberg adjustment)

## Examples

```
python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- The significant reaction set after 5% FDR correction must contain ACONT, ASPTA (forward), RPI, and RPE
- The output table contains both raw empirical p-values and BH-adjusted p-values per reaction, matching the concordanceAnalysis output schema ('the pvalues and the adjusted pvalues')
- The kappa null distribution uses the same 'linear' weighting as the observed concordance analysis
- Each empirical p-value lies in [0,1] and is the fraction of 1000 bootstrap null kappa values at least as extreme as the observed score
- Reactions retained are exactly those with adjusted p-value below 0.05

## Limitations

- The empirical p-value depends on the 1000-resample bootstrap of the RPS pairwise-variation distribution, so resolution is limited to 1/1000 and results may vary slightly across runs
- The validation expectation (ACONT, ASPTA forward, RPI, RPE) is specific to the ENGRO2 model and the immortalized normal/cancer breast cell line dataset used in the article; other models/datasets will yield different significant sets
- The pipeline, including this statistical step, was tested with the GLPK solver and specific Python environment files (integrate_env.txt), so reproducing it requires that environment

## Evidence

- [readme] Concordance step computes kappa with linear weighting and outputs adjusted p-values: "weight: weight on the Kappa Cohen. Default value: 'linear'"
- [readme] README confirms the concordanceAnalysis output includes p-values and adjusted p-values: "A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues"
- [methods] The article's methods list the concordanceAnalysis script usage that precedes this statistical testing: "Usage: `python pipeline/concordanceAnalysis.py`"
- [abstract] The article establishes INTEGRATE's purpose of discriminating regulation levels, the context in which concordance statistics are computed: "We discriminate fluxes regulated at the metabolic and/or gene expression level by intersecting these two output datasets"
