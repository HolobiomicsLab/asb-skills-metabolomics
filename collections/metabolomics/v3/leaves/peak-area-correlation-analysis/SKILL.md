---
name: peak-area-correlation-analysis
description: 'Use when a matched feature table with peak areas from PeakBot and XCMS
  (e.g. from a PHM tool comparison) must be compared: regression R2, trimming of dissimilar
  features, and derivation of the profile-to-centroid median area factor.'
when_to_use_negative:
- Input is raw LC-HRMS profile-mode data that has not yet been peak-picked and grouped
  into a feature table — peak detection with PeakBot must run first
- The two peak-area sets are not matched feature-by-feature (e.g. unaligned peak lists),
  since correlation requires per-feature pairing
- The goal is training or applying a PeakBot CNN model rather than post-hoc comparison
  of two peak-area tables
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PeakBot
  role: Source of the profile-mode peak areas being correlated against XCMS centroid-mode
    areas
  repo: https://github.com/christophuv/PeakBot
- name: Python
  role: Environment for the regression (numpy.polyfit / scipy.stats.linregress), 5%/5%
    trimming, ratio statistics, and CSV export
- name: XCMS
  role: Reference tool whose centroid-mode peak areas serve as the comparison baseline
    in the PHM dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:e1f0f6dcbb03a9bf73c0615df0282dbd3b9f5b941d6b761b9a31d7afddebad60
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - PeakBot
  - Python
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-area-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between peak areas from two peak-picking approaches (PeakBot profile-mode vs XCMS centroid-mode) on matched LC-HRMS features via linear regression, symmetric 5%/5% outlier trimming, and a profile-versus-centroid area-factor statistic.

## When to use

Use when a matched feature table with peak areas from PeakBot and XCMS (e.g. from a PHM tool comparison) must be compared: regression R2, trimming of dissimilar features, and derivation of the profile-to-centroid median area factor.

## When NOT to use

- Input is raw LC-HRMS profile-mode data that has not yet been peak-picked and grouped into a feature table — peak detection with PeakBot must run first
- The two peak-area sets are not matched feature-by-feature (e.g. unaligned peak lists), since correlation requires per-feature pairing
- The goal is training or applying a PeakBot CNN model rather than post-hoc comparison of two peak-area tables

## Inputs

- Matched feature peak-area table from the PHM tool comparison (PeakBot and XCMS areas per aligned feature)
- Feature identifiers with relative area differences for ranking features prior to trimming

## Outputs

- comparison_stats.csv containing feature-level PeakBot/XCMS areas, all-feature R2 (0.808) and trimmed R2 (0.977), and the median area factor with interquartile range (9.96, IQR 9.42–10.78)

## How to apply

Load the matched feature peak-area table from the PHM tool comparison and run a linear regression of PeakBot peak areas against XCMS peak areas in Python (numpy.polyfit or scipy.stats.linregress), computing the coefficient of determination; for all features the reported R2 is 0.808. Then rank features by relative area difference, omit the bottom and top 5% most dissimilar features (10% total), and recompute the fit — the reported trimmed R2 is 0.977, so deviations from these values flag processing errors. Next compute the per-feature ratio of XCMS-to-PeakBot (profile-versus-centroid-mode) areas, deriving the median area factor and its interquartile range; the reported median is 9.96 with IQR 9.42–10.78, reflecting that profile-mode areas are systematically larger than centroid-mode areas. Finally write the feature-level areas, both R2 values, and the area-factor statistics to comparison_stats.csv.

## Related tools

- **PeakBot** (Source of the profile-mode peak areas being correlated against XCMS centroid-mode areas) — https://github.com/christophuv/PeakBot
- **Python** (Environment for the regression (numpy.polyfit / scipy.stats.linregress), 5%/5% trimming, ratio statistics, and CSV export)
- **XCMS** (Reference tool whose centroid-mode peak areas serve as the comparison baseline in the PHM dataset)

## Examples

```
from scipy.stats import linregress; res = linregress(xcms_areas, peakbot_areas); print(res.rvalue**2)
```

## Evaluation signals

- All-feature linear regression of PeakBot vs XCMS areas reproduces R2 = 0.808
- After omitting the bottom and top 5% most dissimilar features (10% total), recomputed R2 = 0.977
- Median XCMS-to-PeakBot area ratio is 9.96 with interquartile range 9.42–10.78
- comparison_stats.csv contains per-feature areas, both R2 values, and the area-factor statistics

## Limitations

- The 10% most dissimilar features (bottom/top 5% by relative area difference) are excluded from the trimmed correlation, so the improved R2 of 0.977 does not describe all features
- Profile-mode (XCMS) areas are systematically larger than centroid-mode (PeakBot) areas — a median factor of 9.96 must be applied before direct area comparison
- Agreement statistics are only as good as the feature matching between the two tools in the PHM comparison

## Evidence

- [other] PeakBot and XCMS peak areas correlate linearly; after omitting the bottom/top 5% of features with too dissimilar peak areas (10% total), the R2 value improved from 0.808 to 0.977.: [evidence span withheld — anchor status: not_found]
- [other] Perform a linear regression of PeakBot peak areas against XCMS peak areas in Python (e.g. with numpy.polyfit or scipy.stats.linregress) and compute the coefficient of determination: [evidence span withheld — anchor status: not_found]
- [readme] The main functions to group detected features from several samples are `peakbot.cuda.KNNalignFeatures` for aligning the detected features: [evidence span withheld — anchor status: external_doc]
