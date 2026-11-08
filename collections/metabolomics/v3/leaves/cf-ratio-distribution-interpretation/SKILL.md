---
name: cf-ratio-distribution-interpretation
description: Use when diagnosing why FSA rank-1 formula accuracy differs across datasets
  by comparing spectral-feature distributions (fragment count, peak-distance SD, CF
  ratio, top-2 score gap) between R1 and R2+ assignments.
when_to_use_negative:
- Features are normally distributed and a parametric comparison (t-test/ANOVA) would
  be more appropriate than the non-parametric Wilcoxon rank-sum test
- Formulas were ranked by tools that use structural information (e.g., Buddy or SIRIUS),
  where FSA's CF-ratio-based feature interpretation does not apply
- Spectra contain elements beyond C, H, O, N, S, and P, which FSA has not been validated
  on
tools:
- name: FSA
  role: Source of the ranked formula assignments and per-spectrum features (fragment
    count, peak-distance SD, CF ratio, top-two score gap) whose R1/R2+ distributions
    are compared
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:a036d32c3808980efc2f95e2615fd76f13f8e73e0c0ce5cf4d56b1aba85e6ff7
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
yes_rate: 0.8
n_claims: 10
---

# cf-ratio-distribution-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets the corrected-fragment (CF) ratio and companion spectral features (fragment count, inter-peak-distance SD, top-two score gap) as discriminators between correctly-ranked (R1) and incorrectly-ranked (R2+) formula assignments produced by FSA, using non-parametric group comparison and boxplot visualization per dataset.

## When to use

Use when diagnosing why FSA rank-1 formula accuracy differs across datasets by comparing spectral-feature distributions (fragment count, peak-distance SD, CF ratio, top-2 score gap) between R1 and R2+ assignments.

## When NOT to use

- Features are normally distributed and a parametric comparison (t-test/ANOVA) would be more appropriate than the non-parametric Wilcoxon rank-sum test
- Formulas were ranked by tools that use structural information (e.g., Buddy or SIRIUS), where FSA's CF-ratio-based feature interpretation does not apply
- Spectra contain elements beyond C, H, O, N, S, and P, which FSA has not been validated on

## Inputs

- Per-spectrum FSA result records containing fragment count, inter-peak-distance SD, CF ratio, top-two score gap, and rank status (R1 vs R2+)
- FSA outputs for the HMDB_QTOF, NPL_QTOF, and LIPOT datasets

## Outputs

- Significance tables of Wilcoxon rank-sum test outcomes per dataset and per feature, with group medians and quartiles
- Boxplots of each spectral feature grouped by R1 vs R2+ rank status, annotated with significance (mirroring Supplementary Figs S9-S12)
- CF-ratio narrative explaining low accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT

## How to apply

Load per-spectrum feature records from the FSA result files for datasets such as HMDB_QTOF, NPL_QTOF, and LIPID_OT, splitting each feature into R1 vs R2+ groups. Because features like fragment numbers are not normally distributed, apply two-sided Wilcoxon rank-sum tests rather than t-tests, recording p-values alongside group medians/quartiles in a significance table. Generate boxplots of each feature grouped by rank status (mirroring Supplementary Figs S9-S12) with significance annotations. Focus interpretation on the CF-ratio distributions across HMDB_QTOF, NPL_QTOF, and LIPID_OT, since CF-ratio behavior is cited as the explanation for low rank-1 accuracy in those datasets; low rank-1 accuracy overall reflects that FSA does not use structural information and is not yet comparable to tools like Buddy or SIRIUS.

## Related tools

- **FSA** (Source of the ranked formula assignments and per-spectrum features (fragment count, peak-distance SD, CF ratio, top-two score gap) whose R1/R2+ distributions are compared) — https://github.com/kslynn128171/FSA

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(cf_ratio_R1, cf_ratio_R2plus, alternative='two-sided')
```

## Evaluation signals

- Reproduced Wilcoxon rank-sum p-values and group distributions match the patterns reported in Supplementary Figs S9-S12
- Fragment-count comparison correctly reports non-significance (features are not normally distributed; p < 0.01 threshold context in the Wilcoxon test narrative)
- CF-ratio distribution comparison reproduces the explanation for low accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT as stated in the results/discussion text
- Boxplots show annotated significance consistent with the significance tables
- Output figure and table files are produced for every dataset-by-feature combination analyzed

## Limitations

- Fragment counts in R1 spectra were mostly higher than in R2+ spectra but were not significantly different under the Wilcoxon rank sum test, so fragment count alone is not a reliable discriminator
- FSA's rank-1 accuracy is not comparable to state-of-the-art tools such as Buddy and SIRIUS because it has not utilized structural information
- Few chemical rules have been implemented to rank formulas or filter out unfeasible ones, limiting CF-ratio interpretability in certain compound species and adduct cases
- FSA has only been validated on compounds composed of C, H, O, N, S, and P

## Evidence

- [supplementary] Fragment counts in R1 spectra were mostly higher than in R2+ spectra, but the Wilcoxon rank sum test found they were not significantly different: [evidence span withheld — anchor status: not_found]
- [discussion] the rank #1 accuracy of the computed formulas was not yet comparable: "the rank #1 accuracy of the computed formulas was not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS"
- [readme] Metabolite identification using fragment subset analysis: "Metabolite identification using fragment subset analysis"
