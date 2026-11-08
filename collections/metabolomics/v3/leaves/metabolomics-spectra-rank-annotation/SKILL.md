---
name: metabolomics-spectra-rank-annotation
description: Use when a per-spectrum feature table with formula-rank labels (R1 vs
  R2+) is available from FSA output and the goal is to test which spectral features
  discriminate correct from incorrect rank#1 formula annotations.
when_to_use_negative:
- Spectra have not been processed by FSA, so no formula-rank labels (R1/R2+) or score
  gaps exist to group features by
- Fragment-count or other feature distributions are verified normal and a two-sided
  Wilcoxon rank-sum test would be underpowered relative to a t-test
- The question is structural identification accuracy rather than formula-ranking feature
  discrimination — FSA accuracy is explicitly 'not yet comparable to that of state-of-the-art
  tools such as Buddy and SIRIUS' and does not use structural information
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: FSA
  role: Produces the ranked formula assignments and per-spectrum scoring outputs (score
    gaps, rank statuses, fragment subsets) whose features are statistically compared
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:cca5fc13fd5e57ff28103bde372b33e127a3e55bd526f35a7881ba355b8e3e3a
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolomics-spectra-rank-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically compares per-spectrum features (fragment count, inter-peak-distance SD, CF ratio, top-two score gap) between correctly ranked (R1) and mis-ranked (R2+) formula candidates from FSA MS/MS identification runs, to identify which spectral characteristics predict correct rank#1 formula assignment.

## When to use

Use when a per-spectrum feature table with formula-rank labels (R1 vs R2+) is available from FSA output and the goal is to test which spectral features discriminate correct from incorrect rank#1 formula annotations.

## When NOT to use

- Spectra have not been processed by FSA, so no formula-rank labels (R1/R2+) or score gaps exist to group features by
- Fragment-count or other feature distributions are verified normal and a two-sided Wilcoxon rank-sum test would be underpowered relative to a t-test
- The question is structural identification accuracy rather than formula-ranking feature discrimination — FSA accuracy is explicitly 'not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS' and does not use structural information

## Inputs

- Per-spectrum feature records from FSA result files: fragment count, inter-peak-distance SD, CF ratio, top-two-score gap
- Formula-rank status labels per spectrum (R1 = correct rank#1 formula, R2+ = mis-ranked)
- Dataset identifiers (e.g. HMDB_QTOF, NPL_QTOF, LIPID_OT from the eight FSA test datasets)

## Outputs

- Wilcoxon rank-sum test p-values per feature per dataset (two-sided)
- Significance tables with group medians and quartiles per dataset and feature
- Annotated boxplots of each feature grouped by R1 vs R2+ (mirroring Supplementary Figs S9–S12)
- Dataset-level CF-ratio distribution comparison explaining low-accuracy datasets

## How to apply

Load per-spectrum feature records — fragment count, inter-peak-distance SD, CF ratio, and top-two-score gap — from the FSA result files (e.g. HMDB_QTOF, NPL_QTOF, LIPID_OT) and split each feature into R1 vs R2+ groups by formula-rank status. Because fragment numbers are not normally distributed (Shapiro-type check motivating nonparametric testing), run two-sided Wilcoxon rank-sum tests per feature per dataset rather than t-tests. Compute group medians and quartiles and assemble per-dataset significance tables. Generate annotated boxplots of each feature grouped by rank status to mirror Supplementary Figs S9–S12. Finally, characterize the CF-ratio distributions across datasets with low rank#1 accuracy, since CF ratio provides the explanatory narrative for poor accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT.

## Related tools

- **FSA** (Produces the ranked formula assignments and per-spectrum scoring outputs (score gaps, rank statuses, fragment subsets) whose features are statistically compared) — https://github.com/kslynn128171/FSA

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(r1_features['top2_gap'], r2p_features['top2_gap'], alternative='two-sided')
```

## Evaluation signals

- Wilcoxon rank-sum p-values reproduce the reported pattern: fragment counts higher in R1 but not significantly different (p-value context: fragment numbers were not normally distributed)
- Boxplots visually match Supplementary Figs S9–S12 patterns for feature-by-rank-status groupings
- Significance tables report medians/quartiles, not means, consistent with non-normal feature distributions
- CF-ratio comparison identifies the low-accuracy datasets (HMDB_QTOF, NPL_QTOF, LIPID_OT) consistent with the article's CF-ratio narrative
- Feature records are split cleanly into mutually exclusive R1 and R2+ groups with no spectra double-counted

## Limitations

- Fragment counts were not normally distributed, so parametric significance tests are inappropriate for this feature
- Results are dataset-specific; CF-ratio explanations were given for low accuracy in HMDB_QTOF, NPL_QTOF, and LIPID_OT and may not transfer to other datasets
- FSA itself implements few chemical rules and 'rank #1 accuracy ... was not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS', so discriminative features reflect FSA's scoring, not universal spectral quality
- FSA was validated only on compounds composed of C, H, O, N, S, and P; features for other element spaces are unvalidated

## Evidence

- [results] Fragment counts in R1 spectra were mostly higher than in R2+ spectra, but the Wilcoxon rank sum test found they were not significantly different: [evidence span withheld — anchor status: not_found]
