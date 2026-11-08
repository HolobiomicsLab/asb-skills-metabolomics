---
name: lc-ms-metabolomics-peak-filtering
description: Use when a detected chromatographic peak list from LC/MS metabolomics
  data must be quality-filtered before downstream statistical analysis, and true positives
  can be validated against a ground-truth peak list.
when_to_use_negative:
- Input is still raw LC/MS profile data with no detected peak list — this skill operates
  post peak detection, after the acute EIC extraction, EIC quality evaluation/filtering,
  and peak detection modules
- The goal is evaluating the quality of extracted ion chromatograms (EICs) themselves
  (e.g., EICs with high background signals or a single peak), which is handled by
  the MCQ index / global zigzag index EIC-evaluation method instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: One of three open-source peak detectors used to define the ground truth peak
    list (R; MatchedFilter and centWave)
- name: MZmine
  role: One of three open-source peak detectors used to define the ground truth peak
    list (Java; centroidPicker)
- name: MAVEN
  role: One of three open-source peak detectors used to define the ground truth peak
    list (C++; machine learning-based peak filtering)
- name: Matlab
  role: Implementation environment for the four-module processing program that computes
    the six peak quality metric values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:e70c73ec9ba96a7617f6110ef32636d58cd2a44d42c628b4bc76d5a01a32d64c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-ms-metabolomics-peak-filtering

## Summary

Filters 'bad' chromatographic peaks from LC/MS metabolomics data by scoring each detected peak against six quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) and applying cutoff thresholds, combining metrics for higher Recall/Precision/F-Score than any single metric.

## When to use

Use when a detected chromatographic peak list from LC/MS metabolomics data must be quality-filtered before downstream statistical analysis, and true positives can be validated against a ground-truth peak list.

## When NOT to use

- Input is still raw LC/MS profile data with no detected peak list — this skill operates post peak detection, after the acute EIC extraction, EIC quality evaluation/filtering, and peak detection modules
- The goal is evaluating the quality of extracted ion chromatograms (EICs) themselves (e.g., EICs with high background signals or a single peak), which is handled by the MCQ index / global zigzag index EIC-evaluation method instead

## Inputs

- Detected chromatographic peak list with six per-peak quality metric values (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) computed by the Matlab four-module processing program
- Ground truth peak list (peaks detected by at least two of XCMS, MZmine, and MAVEN, matched with 0.05 Da m/z and 5 s RT tolerance)
- LC/MS profile dataset from an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer (source of the peaks)

## Outputs

- peak_metric_threshold_sweep_results.csv with per-metric and combined Recall, Precision, and F-Score values
- F-Score versus threshold curves for the six peak quality metrics
- Filtered chromatographic peak list retained after applying the joint metric thresholds

## How to apply

Run the detected peak list through peak quality evaluation after chromatographic peak detection, scoring each peak on six metrics: sharpness, Gaussian similarity, SNR, peak significance level, TPASR (triangle peak area similarity ratio), and local zigzag index. Build a ground truth peak list by intersecting detections from XCMS (R, MatchedFilter/centWave), MZmine (Java, centroidPicker), and MAVEN (C++ ML-based filtering), counting a peak as true only when at least two tools detect it (0.05 Da m/z tolerance, 5 s RT tolerance). Bypass the EIC quality filtering module, sweep each metric's cutoff threshold individually and match retained peaks against ground truth via TP(X)=N(X∩TruePeaklist) to compute Recall, Precision, and F-Score on the whole dataset. Then combine all six metrics at the reported joint thresholds — Sharpness_Th=2.0, Gaussian_Similarity_Th=0.6, SNR_Th=1.3, Peak_Significance_Th=1.2, TPASR_Th=0.8, Zigzag index=0.9 — which should yield F-Score 0.6601 (Recall 0.7076, Precision 0.6186), exceeding any single metric (max single-metric F-Score < 0.6). Sharpness, peak significance level, and zigzag index are expected to be the strongest individual metrics due to the zigzag nature of LC/MS chromatographic peaks.

## Related tools

- **XCMS** (One of three open-source peak detectors used to define the ground truth peak list (R; MatchedFilter and centWave))
- **MZmine** (One of three open-source peak detectors used to define the ground truth peak list (Java; centroidPicker))
- **MAVEN** (One of three open-source peak detectors used to define the ground truth peak list (C++; machine learning-based peak filtering))
- **Matlab** (Implementation environment for the four-module processing program that computes the six peak quality metric values)

## Evaluation signals

- Combined-metric run reproduces Recall = 0.7076, Precision = 0.6186, and F-Score = 0.6601
- Maximum single-metric F-Score remains below 0.6, confirming the combined metrics outperform every individual metric
- F-Score maxima for individual metrics occur for sharpness, peak significance level, and zigzag index
- True positives are computed strictly as TP(X)=N(X∩TruePeaklist) against peaks detected by at least two of the three tools
- All six joint thresholds applied exactly: Sharpness_Th=2.0, Gaussian_Similarity_Th=0.6, SNR_Th=1.3, Peak_Significance_Th=1.2, TPASR_Th=0.8, Zigzag index=0.9

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task; the reported joint thresholds are dataset-specific
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository provided, and no license is specified for the software
- A machine learning-based automatic peak classification (e.g., SVM) is not yet implemented because a curated training set of good and bad representative peaks does not yet exist

## Evidence

- [abstract] combined metrics outperform singles: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [abstract] six metrics and strongest three: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] ground truth definition: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] threshold selection caveat: "setting an optimal cutoff threshold for each metric still is a challenge task"
