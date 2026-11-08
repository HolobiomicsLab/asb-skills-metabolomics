---
name: chromatographic-feature-matching
description: Use when two peak-detection tools have been run on the same LC-HRMS samples
  and you must match their detected features, compare peak areas, and quantify correlation
  (R2) and systematic area-scaling factors.
when_to_use_negative:
- Input is centroided-only LC-HRMS data — PeakBot's peak-picking operates on profile-mode
  data and its area factor quantifies profile-versus-centroid differences
- You need to train a new PeakBot CNN model rather than compare already-detected features
  between tools
- Features have not yet been matched across the two tools and no paired peak-area
  table exists
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Machine-learning peak-picking tool whose detected feature peak areas are matched
    and correlated against XCMS areas
  repo: https://github.com/christophuv/PeakBot
- name: Python 3.8
  role: Environment in which the linear regression, feature ranking/trimming, and
    area-factor computation are performed
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Package implementing the PeakBot CNN model whose peak outputs (peak-type,
    -center, -bounding-box) feed the comparison
  repo: https://www.tensorflow.org/
- name: XCMS
  role: Reference peak-picker whose peak areas provide the comparison baseline for
    the correlation analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:9a1d320f8db2954b0c23fae9ec58d46674024ffad0e02b7334329a89d9331f76
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
  - Python 3.8
  - TensorFlow 2.5.0
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# chromatographic-feature-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Match and compare chromatographic features detected by PeakBot (machine-learning peak-picking in LC-HRMS profile-mode data) against a second peak-picker such as XCMS, quantifying agreement between the two tools' peak areas.

## When to use

Use when two peak-detection tools have been run on the same LC-HRMS samples and you must match their detected features, compare peak areas, and quantify correlation (R2) and systematic area-scaling factors.

## When NOT to use

- Input is centroided-only LC-HRMS data — PeakBot's peak-picking operates on profile-mode data and its area factor quantifies profile-versus-centroid differences
- You need to train a new PeakBot CNN model rather than compare already-detected features between tools
- Features have not yet been matched across the two tools and no paired peak-area table exists

## Inputs

- Peak-area table of matched features from the PHM dataset comparison containing paired PeakBot and XCMS peak areas
- LC-HRMS profile-mode chromatograms processed by PeakBot (feature detection via peakbot.cuda.preProcessChromatogram and peakbot.runPeakBot)
- Feature list or data matrix from PeakBot grouping (peakbot.cuda.KNNalignFeatures and peakbot.cuda.groupFeatures)

## Outputs

- comparison_stats.csv with feature-level PeakBot and XCMS peak areas, R2 values, and area-factor statistics
- Per-feature XCMS-to-PeakBot area ratios with median area factor and IQR
- Linear-regression fit (slope/intercept) and coefficient of determination for matched feature peak areas

## How to apply

Load the table of matched feature peak areas for PeakBot and XCMS from the PHM tool comparison. Fit a linear regression of PeakBot peak areas against XCMS peak areas (numpy.polyfit or scipy.stats.linregress) and compute the coefficient of determination, confirming the reported R2 = 0.808 for all features. Rank features by relative area difference, omit the bottom and top 5% most dissimilar features (10% total), and recompute R2, which should rise to the reported 0.977. Then compute the per-feature ratio of XCMS-to-PeakBot peak areas — this ratio reflects the profile-versus-centroid-mode area difference — and derive the median area factor and its interquartile range, expected at a median of 9.96 (IQR 9.42-10.78). Write feature-level areas, computed R2 values, and area-factor statistics to comparison_stats.csv.

## Related tools

- **PeakBot** (Machine-learning peak-picking tool whose detected feature peak areas are matched and correlated against XCMS areas) — https://github.com/christophuv/PeakBot
- **Python 3.8** (Environment in which the linear regression, feature ranking/trimming, and area-factor computation are performed) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Package implementing the PeakBot CNN model whose peak outputs (peak-type, -center, -bounding-box) feed the comparison) — https://www.tensorflow.org/
- **XCMS** (Reference peak-picker whose peak areas provide the comparison baseline for the correlation analysis)

## Examples

```
from scipy.stats import linregress; r = linregress(xcms_areas, peakbot_areas); print(r.rvalue**2)  # expect ~0.808
```

## Evaluation signals

- Linear regression of PeakBot vs XCMS peak areas over all features reproduces R2 = 0.808
- After omitting the bottom/top 5% of features with most dissimilar peak areas (10% total), R2 improves to the reported 0.977
- Median XCMS-to-PeakBot peak-area ratio equals 9.96 with interquartile range 9.42-10.78
- Every retained feature has paired non-missing peak areas from both tools in the matched-feature table
- comparison_stats.csv contains feature-level areas, computed R2 values, and area-factor statistics

## Limitations

- PeakBot is designed for profile-mode LC-HRMS data; comparisons to centroid-mode tools such as XCMS are confounded by a systematic area factor (median 9.96, IQR 9.42-10.78) that must be reported rather than ignored
- The 5% bottom/top trimming of most dissimilar features is a post-hoc filtering step, so the improved R2 = 0.977 applies only to the central 90% of features
- Training a PeakBot model requires at least 100 reference features and correctly set profile-mode parameters (noiseLevel, minIntensity, minRT/maxRT, RTpeakWidth, intra/inter-scan ppm differences) before features can be matched

## Evidence

- [intro] PeakBot is a python package for peak-picking in LC-HRMS profile mode data.: "PeakBot is a python package for peak-picking in LC-HRMS profile mode data."
- [abstract] PeakBot achieved high accuracy: "In training and independent validation datasets used for development, PeakBot achieved a high performance with respect to discriminating between chromatographic peaks and background signals"
- [readme] KNN grouping functions: [evidence span withheld — anchor status: external_doc]
- [readme] Detection functions: [evidence span withheld — anchor status: external_doc]
- [abstract] Minimum reference features: "For training the machine-learning model a minimum of 100 reference features are needed to learn their characteristics to achieve high-quality peak-picking results"
