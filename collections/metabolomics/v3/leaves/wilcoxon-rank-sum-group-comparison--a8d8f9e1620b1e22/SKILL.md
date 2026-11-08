---
name: wilcoxon-rank-sum-group-comparison
description: 'Use when comparing a numeric spectral or ranking feature between two
  independent groups (e.g., correct rank #1 vs lower-ranked FSA formula assignments)
  where the feature is non-normally distributed, so a rank-based test like Wilcoxon
  rank-sum with medians/quartiles is required.'
when_to_use_negative:
- The feature is normally distributed and a two-sample t-test is statistically appropriate
  — the rank-sum test is chosen here specifically because fragment numbers were not
  normally distributed
- The comparison involves paired/matched observations rather than two independent
  groups (Wilcoxon rank-sum assumes independence)
- Input is raw MS/MS peak lists rather than already-computed per-spectrum feature
  records from an FSA ranking run
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: FSA
  role: Produces the per-spectrum formula-ranking results and feature records (fragment
    count, CF ratio, etc.) whose R1 vs R2+ groups are compared with Wilcoxon rank-sum
    tests
  repo: kslynn128171/FSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:cb7fa207c9267efdd35cbb77c4822e79e0ff01e8e4a83e3fa307278c16628ec7
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# wilcoxon-rank-sum-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Two-sided Wilcoxon rank-sum testing of per-spectrum spectral features (fragment count, inter-peak-distance SD, CF ratio, top-two-score gap) split by formula-ranking outcome (rank #1 vs rank 2+) to identify which features statistically discriminate correct formula assignments made by FSA.

## When to use

Use when comparing a numeric spectral or ranking feature between two independent groups (e.g., correct rank #1 vs lower-ranked FSA formula assignments) where the feature is non-normally distributed, so a rank-based test like Wilcoxon rank-sum with medians/quartiles is required.

## When NOT to use

- The feature is normally distributed and a two-sample t-test is statistically appropriate — the rank-sum test is chosen here specifically because fragment numbers were not normally distributed
- The comparison involves paired/matched observations rather than two independent groups (Wilcoxon rank-sum assumes independence)
- Input is raw MS/MS peak lists rather than already-computed per-spectrum feature records from an FSA ranking run

## Inputs

- Per-spectrum FSA result records containing fragment count, inter-peak-distance SD, CF ratio, top-two-score gap, and rank status (R1 vs R2+)
- Dataset designations (e.g., HMDB_QTOF, NPL_QTOF, LIPID_OT) to stratify tests and plots

## Outputs

- Significance tables of two-sided Wilcoxon rank-sum test p-values per dataset and per feature
- Group medians/quartiles per feature and rank-status group
- Boxplots of each feature grouped by rank status (annotated with significance), mirroring Supplementary Figs S9-S12

## How to apply

Load per-spectrum feature records (fragment count, inter-peak-distance SD, CF ratio, top-two-score gap, and rank status R1 vs R2+) from FSA result files for each dataset (e.g., HMDB_QTOF, NPL_QTOF, LIPID_OT). First check normality — in this analysis fragment numbers were not normally distributed, which mandates the rank-based Wilcoxon rank-sum test instead of a t-test. Split each feature into R1 and R2+ groups, run two-sided Wilcoxon rank-sum tests, and record p-values per dataset and per feature. Compute group medians and quartiles and assemble significance tables; then generate boxplots per feature grouped by rank status, annotated with significance, to mirror Supplementary Figs S9-S12. Interpret non-significant results carefully (e.g., fragment counts trended higher in R1 spectra but Wilcoxon gave p < 0.01 threshold not met) and characterize explanatory features such as CF-ratio distributions when narrating low-accuracy datasets.

## Related tools

- **FSA** (Produces the per-spectrum formula-ranking results and feature records (fragment count, CF ratio, etc.) whose R1 vs R2+ groups are compared with Wilcoxon rank-sum tests) — kslynn128171/FSA

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(r1_features['fragment_count'], r2p_features['fragment_count'], alternative='two-sided')
```

## Evaluation signals

- Reproduced Wilcoxon rank-sum p-values match the patterns reported in Supplementary Figs S9-S12
- Group median/quartile ordering matches reported trends (e.g., fragment counts mostly higher in R1 than R2+ spectra)
- Normality pre-check confirms feature non-normality, justifying rank-sum over parametric testing
- CF-ratio distributions across HMDB_QTOF, NPL_QTOF, and LIPID_OT reproduce the narrative used to explain low rank-#1 accuracy in those datasets
- Significance tables cover every dataset x feature combination with recorded two-sided p-values

## Limitations

- Non-significant differences can still show visible trends: fragment counts were mostly higher in R1 spectra yet Wilcoxon found no significant difference
- Rank-sum results are affected by small group sizes and skewed feature distributions common in MS/MS spectral features
- Features explain but do not causally establish ranking accuracy; CF-ratio behavior only correlates with low accuracy in specific datasets

## Evidence

- [results] Fragment counts trend but do not discriminate significantly: [evidence span withheld — anchor status: not_found]
- [supplementary] Workflow: split and test features: [evidence span withheld — anchor status: not_found]
- [supplementary] Outputs mirror supplementary figures: [evidence span withheld — anchor status: not_found]
