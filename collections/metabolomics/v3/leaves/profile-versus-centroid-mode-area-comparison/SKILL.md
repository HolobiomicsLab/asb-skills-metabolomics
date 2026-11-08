---
name: profile-versus-centroid-mode-area-comparison
description: Use when a matched feature table with peak areas from two peak-picking
  tools (e.g. PeakBot and XCMS) is available and the goal is to quantify correlation
  (R2) and the systematic profile-versus-centroid area scaling factor.
when_to_use_negative:
- The input is raw LC-HRMS profile-mode chromatograms rather than a matched feature
  peak-area table — peak detection/grouping must be run first (e.g. peakbot.cuda.groupFeatures
  or XCMS).
- Features are not matched across the two tools; unmatched feature lists cannot yield
  per-feature area ratios or a meaningful regression.
- Only centroid-mode data is available, since the skill quantifies the profile-versus-centroid-mode
  area scaling between PeakBot (profile) and XCMS (centroid).
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Source of the profile-mode peak areas being compared; its grouped feature
    table supplies one side of the regression and area-factor ratios
  repo: https://github.com/christophuv/PeakBot
- name: Python
  role: Implemented in python (3.8); used to run the linear regression (numpy.polyfit
    or scipy.stats.linregress), trimming, and area-factor statistics
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:b42e729d2d3a34b646edf88c830f69f41745326182c152b0aeb24b20081b6783
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# profile-versus-centroid-mode-area-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitative comparison of matched feature peak areas from PeakBot (profile-mode LC-HRMS peak-picking) and XCMS via linear regression and robust trimming, plus estimation of the profile-versus-centroid-mode area factor. It verifies how closely PeakBot-derived peak areas agree with XCMS and quantifies the systematic area scaling between the two acquisition modes.

## When to use

Use when a matched feature table with peak areas from two peak-picking tools (e.g. PeakBot and XCMS) is available and the goal is to quantify correlation (R2) and the systematic profile-versus-centroid area scaling factor.

## When NOT to use

- The input is raw LC-HRMS profile-mode chromatograms rather than a matched feature peak-area table — peak detection/grouping must be run first (e.g. peakbot.cuda.groupFeatures or XCMS).
- Features are not matched across the two tools; unmatched feature lists cannot yield per-feature area ratios or a meaningful regression.
- Only centroid-mode data is available, since the skill quantifies the profile-versus-centroid-mode area scaling between PeakBot (profile) and XCMS (centroid).

## Inputs

- Peak-area table from the PHM tool comparison with matched feature peak areas for PeakBot and XCMS (one row per matched feature)
- Python 3.8 environment with numpy and scipy

## Outputs

- comparison_stats.csv containing feature-level PeakBot/XCMS peak areas, R2 values (all features and after 5%/5% trimming), and XCMS-to-PeakBot area-factor statistics (median and IQR)
- Linear regression R2 values for all matched features and for the trimmed set
- Median profile-versus-centroid-mode area factor with interquartile range

## How to apply

Load the peak-area table from the tool comparison containing matched feature peak areas for PeakBot and XCMS, and fit PeakBot areas against XCMS areas using numpy.polyfit or scipy.stats.linregress in Python 3.8 to obtain the coefficient of determination (expected ~0.808 for all features). Then rank features by relative area difference, omit the bottom and top 5% most dissimilar features (10% total) to reduce the influence of outliers, and recompute R2 (expected ~0.977). Next compute the per-feature XCMS-to-PeakBot area ratio (profile-versus-centroid-mode areas), derive the median area factor and its interquartile range, and check against the reported median of 9.96 (IQR 9.42–10.78). Finally write feature-level areas, computed R2 values, and the area-factor statistics to comparison_stats.csv; a large median ratio reflects that profile-mode areas integrated by PeakBot are systematically larger than centroid-mode XCMS areas.

## Related tools

- **PeakBot** (Source of the profile-mode peak areas being compared; its grouped feature table supplies one side of the regression and area-factor ratios) — https://github.com/christophuv/PeakBot
- **Python** (Implemented in python (3.8); used to run the linear regression (numpy.polyfit or scipy.stats.linregress), trimming, and area-factor statistics) — https://github.com/christophuv/PeakBot

## Examples

```
from scipy.stats import linregress; res = linregress(xcms_areas, peakbot_areas); print(res.rvalue**2)
```

## Evaluation signals

- R2 of the full linear regression of PeakBot vs XCMS peak areas reproduces the reported 0.808.
- After omitting the bottom and top 5% of features by relative area difference (10% total), R2 reproduces the reported 0.977.
- Median XCMS-to-PeakBot area ratio reproduces the reported 9.96 with IQR 9.42–10.78.
- PeakBot vs XCMS peak areas correlate linearly (regression of one against the other is approximately linear before and after trimming).
- comparison_stats.csv contains per-feature areas plus the computed R2 values and area-factor statistics.

## Limitations

- The trimming step discards 10% of features (bottom/top 5% by relative area difference) whose peak areas are too dissimilar, so the improved R2 = 0.977 does not reflect all features.
- The area factor (median 9.96, IQR 9.42–10.78) is dataset- and tool-pairing-specific (PeakBot vs XCMS on the PHM comparison) and should not be assumed universal.
- Feature matching between the two tools must be correct beforehand; mis-matched features would inflate the outlier fraction trimmed in step 3.

## Evidence

- [full_text] PeakBot and XCMS peak areas correlate linearly; after omitting the bottom/top 5% of features with too dissimilar peak areas (10% total), the R2 value improved from 0.808 to 0.977.: [evidence span withheld — anchor status: not_found]
- [readme] This script shows how detected chromatographic peaks can be grouped into a data matrix and a featureML file.: [evidence span withheld — anchor status: external_doc]
