---
name: outlier-trimming-quantile-statistics
description: Use when matched peak-area tables from two peak-detection tools on the
  same LC-HRMS dataset (e.g. PeakBot profile-mode vs XCMS centroid-mode on the PHM
  dataset) need an agreement metric, and a small fraction of features with too dissimilar
  areas must be excluded before reporting correlation.
when_to_use_negative:
- The input is already a cleaned/trimmed correlation table — re-trimming would double-remove
  features and inflate R²
- Features are not matched one-to-one across the two tools, so relative area differences
  cannot be ranked per feature
- The analysis goal is peak detection or CNN model training itself rather than post-hoc
  agreement statistics on peak areas
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3316
tools:
- name: Python
  role: Runs the linear regression (numpy.polyfit / scipy.stats.linregress), the 5%-tail
    trimming of ranked features, and the interquartile-range statistics
- name: PeakBot
  role: Source of the profile-mode peak areas that are correlated against XCMS centroid-mode
    areas and scaled by the derived median area factor
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:568f9c6efeb4f12a7f2feb2b45fc7c2be2ae7659c1697c9b65097ebbc03f0d30
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# outlier-trimming-quantile-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A reproducible statistical procedure for comparing LC-HRMS peak areas from two feature-detection tools: regress PeakBot peak areas against XCMS peak areas, trim the 5% tails of features ranked by relative area difference, and report R² plus the median XCMS-to-PeakBot area factor with its interquartile range.

## When to use

Use when matched peak-area tables from two peak-detection tools on the same LC-HRMS dataset (e.g. PeakBot profile-mode vs XCMS centroid-mode on the PHM dataset) need an agreement metric, and a small fraction of features with too dissimilar areas must be excluded before reporting correlation.

## When NOT to use

- The input is already a cleaned/trimmed correlation table — re-trimming would double-remove features and inflate R²
- Features are not matched one-to-one across the two tools, so relative area differences cannot be ranked per feature
- The analysis goal is peak detection or CNN model training itself rather than post-hoc agreement statistics on peak areas

## Inputs

- Matched feature table of peak areas from PeakBot and XCMS for the same features (e.g. from the PHM dataset comparison)
- Python 3 environment with numpy and scipy

## Outputs

- comparison_stats.csv with feature-level peak areas, computed R² values (all features and trimmed), and area-factor statistics (median, IQR)
- Trimmed feature ranking by relative area difference

## How to apply

Load the matched feature table of PeakBot and XCMS peak areas from the PHM tool comparison. Perform a linear regression of PeakBot areas against XCMS areas in Python (numpy.polyfit or scipy.stats.linregress) and check the coefficient of determination against the reported R² = 0.808 for all features. Rank features by relative area difference, omit the bottom and top 5% most dissimilar features (10% total), and recompute the regression, verifying the reported improvement to R² = 0.977. Then compute the per-feature ratio of XCMS-to-PeakBot (profile-versus-centroid-mode) areas, deriving the median area factor (reported 9.96) and its interquartile range (reported 9.42–10.78). Write feature-level areas, computed R² values, and area-factor statistics to comparison_stats.csv.

## Related tools

- **Python** (Runs the linear regression (numpy.polyfit / scipy.stats.linregress), the 5%-tail trimming of ranked features, and the interquartile-range statistics)
- **PeakBot** (Source of the profile-mode peak areas that are correlated against XCMS centroid-mode areas and scaled by the derived median area factor) — https://github.com/christophuv/PeakBot

## Examples

```
import numpy as np, scipy.stats as st; slope,inter,r,p,se = st.linregress(xcms_areas, peakbot_areas)
```

## Evaluation signals

- R² of the full-feature PeakBot-vs-XCMS regression reproduces the reported 0.808
- R² after omitting the 10% most dissimilar features reproduces the reported 0.977
- Median XCMS-to-PeakBot peak-area ratio is 9.96 with IQR 9.42–10.78
- Exactly 5% of features are dropped from each tail of the relative-area-difference ranking (10% total)
- comparison_stats.csv contains feature-level areas, both R² values, and the area-factor statistics

## Limitations

- The trimmed R² (0.977) is inflated relative to the untrimmed value (0.808); both values should be reported since 10% of features are discarded by construction
- The median area factor (~9.96) is specific to profile-versus-centroid-mode area scaling and must not be reused for comparisons of two tools in the same mode
- Results depend on a correctly matched feature table; mismatches enter the tails and skew the trimming

## Evidence

- [results] Trimming rationale and target R² values: [evidence span withheld — anchor status: not_found]
- [methods] Area-factor computation: [evidence span withheld — anchor status: not_found]
