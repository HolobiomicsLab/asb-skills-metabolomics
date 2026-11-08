---
name: spectral-feature-discrimination-analysis
description: Use when FSA (or similar formula-ranking) result files contain per-spectrum
  rank outcomes plus spectral features, and you need to identify which features statistically
  separate R1 from R2+ assignments.
when_to_use_negative:
- Spectra lack rank outcomes or the four per-spectrum features (fragment count, inter-peak-distance
  SD, CF ratio, top-two-score gap), so no R1/R2+ grouping is possible
- 'You need absolute ranking accuracy of FSA (rank #1/top-5 rates) rather than feature-level
  discrimination between correct and incorrect assignments'
- Features are normally distributed and a parametric comparison (t-test/ANOVA) is
  the appropriate test — the article applies Wilcoxon because fragment numbers are
  not normally distributed
tools:
- name: FSA
  role: Source of the per-spectrum feature records and rank status (R1 vs R2+) that
    feed the Wilcoxon rank-sum comparisons and boxplots
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:260dc12b2afc11fcfcd0c152badf6e6233a077724759190f2013b5cb62ffd7e1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-feature-discrimination-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically discriminates correctly-ranked (rank #1) from incorrectly-ranked (R2+) MS/MS formula assignments by comparing per-spectrum spectral features — fragment count, inter-peak-distance SD, CF ratio, and top-two-score gap — with two-sided Wilcoxon rank-sum tests and annotated boxplots.

## When to use

Use when FSA (or similar formula-ranking) result files contain per-spectrum rank outcomes plus spectral features, and you need to identify which features statistically separate R1 from R2+ assignments.

## When NOT to use

- Spectra lack rank outcomes or the four per-spectrum features (fragment count, inter-peak-distance SD, CF ratio, top-two-score gap), so no R1/R2+ grouping is possible
- You need absolute ranking accuracy of FSA (rank #1/top-5 rates) rather than feature-level discrimination between correct and incorrect assignments
- Features are normally distributed and a parametric comparison (t-test/ANOVA) is the appropriate test — the article applies Wilcoxon because fragment numbers are not normally distributed

## Inputs

- Per-spectrum FSA result records containing fragment count, inter-peak-distance SD, CF ratio, top-two-score gap, and rank status (R1 vs R2+)
- FSA output for datasets such as HMDB_QTOF, NPL_QTOF, and LIPID_OT

## Outputs

- Per-dataset, per-feature significance table of two-sided Wilcoxon rank-sum test p-values with group medians/quartiles
- Boxplots of each spectral feature grouped by rank status (R1 vs R2+) with significance annotations, mirroring Supplementary Figs S9-S12
- CF-ratio distribution comparison explaining low-accuracy datasets

## How to apply

Load per-spectrum feature records (fragment count, inter-peak-distance SD, CF ratio, top-two-score gap, rank status) from the FSA result files for each dataset (e.g., HMDB_QTOF, NPL_QTOF, LIPID_OT). Split each feature into R1 versus R2+ groups and run two-sided Wilcoxon rank-sum tests rather than t-tests, because fragment numbers are not normally distributed (Shapiro-type non-normality, p < 0.01). Compute group medians and quartiles and assemble per-dataset, per-feature significance tables. Generate boxplots of each feature grouped by rank status, annotating significance, to mirror Supplementary Figs S9-S12. Finally, compare CF-ratio distributions across low-accuracy datasets to characterize why fragment CF ratios explain poor rank-#1 accuracy (as reported for HMDB_QTOF, NPL_QTOF, and LIPID_OT).

## Related tools

- **FSA** (Source of the per-spectrum feature records and rank status (R1 vs R2+) that feed the Wilcoxon rank-sum comparisons and boxplots) — https://github.com/kslynn128171/FSA

## Examples

```
python -c "from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(r1_fragcounts, r2p_fragcounts, alternative='two-sided'); print(U, p)"
```

## Evaluation signals

- Reproduced Wilcoxon rank-sum p-values match the significance patterns reported in Supplementary Figs S9-S12
- Fragment-count comparison reproduces the reported result: higher in R1 than R2+ but not statistically significant
- Boxplots show group medians/quartiles consistent with the article's group distributions for each of the four features
- CF-ratio distributions reproduce the narrative explanation given for low rank-#1 accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT
- Feature splitting yields exactly two groups per feature (R1 and R2+) covering all spectra with no unassigned rank status

## Limitations

- Fragment counts, despite being mostly higher in R1 spectra, were not significantly different between rank groups, so fragment count alone cannot predict correct ranking
- FSA's rank #1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS because it does not utilize structural information
- Few chemical rules have been implemented to rank or filter unfeasible formulas, limiting the interpretability of rank status as a quality signal

## Evidence

- [supplementary] Fragment counts in R1 spectra were mostly higher than in R2+ spectra, but the Wilcoxon rank sum test found they were not significantly different: [evidence span withheld — anchor status: not_found]
- [methods] Split each feature into R1 and R2+ groups and run two-sided Wilcoxon rank-sum tests comparing the groups, recording p-values: [evidence span withheld — anchor status: not_found]
- [methods] Load the per-spectrum feature records (fragment count, inter-peak-distance SD, CF ratio, top-two-score gap, rank status R1 vs R2+) from the FSA result files: [evidence span withheld — anchor status: not_found]
- [methods] Generate boxplots of each feature grouped by rank status (R1 vs R2+) to mirror Supplementary Figs S9-S12, annotating significance: [evidence span withheld — anchor status: not_found]
- [results] Characterize the CF-ratio explanations given for the low accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT by comparing CF-ratio distributions across those datasets: [evidence span withheld — anchor status: not_found]
