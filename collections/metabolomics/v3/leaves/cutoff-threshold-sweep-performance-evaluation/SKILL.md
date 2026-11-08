---
name: cutoff-threshold-sweep-performance-evaluation
description: 'Use when filtering detected chromatographic peaks by quality metrics
  and you need to choose defensible cutoff thresholds: you have per-peak metric values
  (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, zigzag index),
  a ground truth peak list, and must quantify the.'
when_to_use_negative:
- When the input is a raw LC/MS profile dataset rather than an already detected peak
  list with per-peak quality metric values — first run the four-module Matlab processing
  pipeline (acute EIC extraction, EIC filtering, peak detection, peak quality evaluation)
  before sweeping cutoffs.
- When no ground truth peak list is available (e.g. no consensus of XCMS/MZmine/MAVEN
  detections); Recall/Precision/F-Score cannot be computed and the sweep produces
  unverifiable threshold rankings.
- For evaluating extracted ion chromatogram (EIC) quality itself using the MCQ index
  or global zigzag index — that is a separate EIC-level evaluation task, not this
  per-peak threshold sweep.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: One of the three tools whose detections form the ground truth peak list used
    to compute TP/FP for the threshold sweep
- name: MZmine
  role: Second tool for the consensus ground truth peak list
- name: MAVEN
  role: Third tool for the consensus ground truth peak list
- name: Matlab
  role: Implements the four-module program that produced the detected peak list and
    its six quality metrics; used to run the cutoff sweeps and compute Recall/Precision/F-Score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:9876c60f2d13e6993110d2b6cfd0d8f2fb6f609b3864f4380c89836ad594c6c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cutoff-threshold-sweep-performance-evaluation

## Summary

Systematically vary quality-metric cutoff thresholds — individually and in combination — over detected LC/MS chromatographic peaks, compute Recall/Precision/F-Score against a ground truth peak list, and select thresholds by maximizing F-Score for peak filtering.

## When to use

Use when filtering detected chromatographic peaks by quality metrics and you need to choose defensible cutoff thresholds: you have per-peak metric values (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, zigzag index), a ground truth peak list, and must quantify the detection-performance trade-off of each threshold setting.

## When NOT to use

- When the input is a raw LC/MS profile dataset rather than an already detected peak list with per-peak quality metric values — first run the four-module Matlab processing pipeline (acute EIC extraction, EIC filtering, peak detection, peak quality evaluation) before sweeping cutoffs.
- When no ground truth peak list is available (e.g. no consensus of XCMS/MZmine/MAVEN detections); Recall/Precision/F-Score cannot be computed and the sweep produces unverifiable threshold rankings.
- For evaluating extracted ion chromatogram (EIC) quality itself using the MCQ index or global zigzag index — that is a separate EIC-level evaluation task, not this per-peak threshold sweep.

## Inputs

- Detected chromatographic peak list with six quality metric values per peak (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index), computed by the four-module Matlab program (acute EIC extraction → EIC quality evaluation and filtering → chromatographic peak detection → peak quality evaluation and filtering)
- Ground truth peak list defined as peaks detected by at least two of XCMS, MZmine, and MAVEN on the same UPLC/MS dataset

## Outputs

- Per-metric F-Score versus threshold curves for the six peak quality metrics
- Recall, Precision, and F-Score values for each single-metric threshold sweep (expected single-metric maxima < 0.6)
- Combined six-metric Recall, Precision, and F-Score (expected Recall = 0.7076, Precision = 0.6186, F-Score = 0.6601)
- Results table (e.g. peak_metric_threshold_sweep_results.csv) listing threshold settings and the three performance scores

## How to apply

Bypass the EIC quality filtering module and select one peak quality metric at a time, sweeping its cutoff threshold while matching each retained peak against the ground truth list (peaks detected by at least two of XCMS, MZmine, and MAVEN, matched with 0.05 Da m/z and 5 s retention-time tolerance) using TP(X) = N(X ∩ TruePeaklist). At each threshold compute Recall, Precision, and F-Score and record the F-Score curve; confirm which metrics reach maxima (sharpness, peak significance level, and zigzag index in this dataset). Then combine all six metrics at the reported joint thresholds — Sharpness_Th = 2.0, Gaussian_Similarity_Th = 0.6, SNR_Th = 1.3, Peak_Significance_Th = 1.2, TPASR_Th = 0.8, Zigzag index = 0.9 — and recompute the three scores. The rationale is that, because of the zigzag nature of LC/MS chromatographic peaks, combining several peak quality metrics is more efficient than any single metric; the combined run must exceed every single-metric maximum, which should stay below an F-Score of 0.6.

## Related tools

- **XCMS** (One of the three tools whose detections form the ground truth peak list used to compute TP/FP for the threshold sweep)
- **MZmine** (Second tool for the consensus ground truth peak list)
- **MAVEN** (Third tool for the consensus ground truth peak list)
- **Matlab** (Implements the four-module program that produced the detected peak list and its six quality metrics; used to run the cutoff sweeps and compute Recall/Precision/F-Score)

## Examples

```
matlab -r "sweep_peak_metrics('detected_peaks.csv','ground_truth_consensus.csv','peak_metric_threshold_sweep_results.csv');"
```

## Evaluation signals

- Combined six-metric run reproduces Recall = 0.7076, Precision = 0.6186, F-Score = 0.6601
- Maximum single-metric F-Score remains smaller than 0.6
- F-Score curves for sharpness, peak significance level, and zigzag index exhibit visible maxima as their cutoffs are swept
- Ground truth matches use 0.05 Da m/z tolerance and 5 seconds retention time tolerance against the ≥2-of-3 consensus list
- Results are written to a single results table (peak_metric_threshold_sweep_results.csv) containing one row per threshold setting with Recall, Precision, and F-Score

## Limitations

- Setting an optimal cutoff threshold for each metric remains an unsolved challenge; reported joint thresholds are dataset- and instrument-specific (Agilent HPLC + Q-TOF Premier data) and may not transfer directly.
- Ground truth is derived from a majority consensus of only three tools (XCMS, MZmine, MAVEN), so errors shared by two tools would be counted as true positives.
- Requires a curated ground truth list; a machine-learning approach (e.g. SVM) that would avoid manual threshold selection needs a training set of good and bad representative peaks that does not yet exist.
- Combining all six metrics was evaluated as a single joint configuration; the article does not exhaustively explore subsets of metric combinations.

## Evidence

- [intro] During data processing, the metric cutoff thresholds were varied, both individually and in combination,: "During data processing, the metric cutoff thresholds were varied, both individually and in combination,"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [abstract] using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
