---
name: eic-quality-threshold-classification
description: Use when filtering extracted ion chromatograms (EICs) from LC/MS metabolomics
  profiles before peak detection — especially when EICs contain high background signals
  or only a single peak, where the MCQ index scores unfairly and cutoff thresholds
  must be tuned.
when_to_use_negative:
- Evaluating already-detected chromatographic peaks after peak detection rather than
  filtering whole EICs before peak detection — the six peak-level metrics (sharpness,
  Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag
  index) address that separate stage.
- The EIC set contains high-background or single-peak profiles and the plan is to
  classify them with the MCQ index alone — the paper shows MCQ cannot fairly evaluate
  these scenarios; use the global zigzag index.
- Training an automatic machine-learning-based peak identifier (e.g., SVM) on curated
  good/bad examples — this skill is metric-plus-threshold classification, not ML-based
  peak identification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: Matlab
  role: Custom data processing program implementing the four sequential modules, including
    EIC quality evaluation and filtering, the MCQ index, and the global zigzag index
    computations
- name: XCMS
  role: R peak detector (MatchedFilter and centWave options) whose peak list contributes
    to the ground truth peaks used for Recall/Precision/F-Score threshold calibration
- name: MZmine
  role: Java peak detector (centroidPicker algorithm) whose peak list contributes
    to the multi-tool ground truth consensus
- name: MAVEN
  role: C++ detector with a machine-learning-based peak filtering method whose peak
    list contributes to the ground truth consensus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:1aa89d707edfa13a56722393f0aeb4b959f4927b071f76894e45daf4432fba8a
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# eic-quality-threshold-classification

## Summary

Classifies extracted ion chromatograms (EICs) as 'good' or 'bad' by scoring each with quality metrics — notably the mass chromatographic quality (MCQ) index and the EIC-related global zigzag index constrained to [0, 4.0] — and applying cutoff thresholds evaluated by Recall, Precision, and F-Score. It removes bad EICs before the downstream time-consuming chromatographic peak detection step of LC/MS metabolomics processing.

## When to use

Use when filtering extracted ion chromatograms (EICs) from LC/MS metabolomics profiles before peak detection — especially when EICs contain high background signals or only a single peak, where the MCQ index scores unfairly and cutoff thresholds must be tuned.

## When NOT to use

- Evaluating already-detected chromatographic peaks after peak detection rather than filtering whole EICs before peak detection — the six peak-level metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) address that separate stage.
- The EIC set contains high-background or single-peak profiles and the plan is to classify them with the MCQ index alone — the paper shows MCQ cannot fairly evaluate these scenarios; use the global zigzag index.
- Training an automatic machine-learning-based peak identifier (e.g., SVM) on curated good/bad examples — this skill is metric-plus-threshold classification, not ML-based peak identification.

## Inputs

- Extracted ion chromatogram (EIC) intensity profiles from an LC/MS profile dataset (e.g. Agilent HPLC interfaced with a Q-TOF Premier mass spectrometer; 611 extracted EICs, with representative EIC_IDs 27, 40, 90, 134, 154, 167 from Table 1)
- Candidate metric cutoff threshold values to vary individually and in combination
- Peak lists from XCMS, MZmine, and MAVEN used to define ground truth peaks for Recall/Precision/F-Score calibration

## Outputs

- Per-EIC quality metric values (MCQ index, EIC-related global zigzag index in [0, 4.0])
- Binary good/bad verdict per EIC under each metric and threshold setting, written as a tab-separated table
- Recall, Precision, and F-Score per cutoff threshold setting (individual and combined)

## How to apply

Run this as module 2 of the paper's four-module Matlab pipeline (acute EIC extraction -> EIC quality evaluation and filtering -> chromatographic peak detection -> peak quality evaluation and filtering), so bad EICs are removed before time-consuming peak detection. Compute the MCQ index per EIC as the similarity of the original mass chromatogram to its smoothed version (spike detection) and to its mean-subtracted version (background detection). Compute the EIC-related global zigzag index from the EIC's first-order derivatives as the sum of squared second-order differences (2*In - In-1 - In+1)^2 over neighboring data points, normalized by N times the squared effective peak intensity (max intensity minus baseline at apex), yielding a value in [0, 4.0]; prefer it over the MCQ index when EICs have high background signals or contain only a single peak. Vary the metric cutoff thresholds individually and in combination, and score each setting by Recall, Precision, and F-Score against ground truth peaks, defined as peaks detected by at least two of XCMS (R; MatchedFilter, centWave), MZmine (Java; centroidPicker), and MAVEN (C++; machine-learning-based peak filtering). Combine several metrics rather than relying on one — the paper found that using several peak quality metrics in combination is more efficient than individual metrics, with sharpness, peak significance level, and zigzag index the best of its six peak-quality metrics — and write per-EIC metric values plus good/bad verdicts to a tab-separated table.

## Related tools

- **Matlab** (Custom data processing program implementing the four sequential modules, including EIC quality evaluation and filtering, the MCQ index, and the global zigzag index computations)
- **XCMS** (R peak detector (MatchedFilter and centWave options) whose peak list contributes to the ground truth peaks used for Recall/Precision/F-Score threshold calibration)
- **MZmine** (Java peak detector (centroidPicker algorithm) whose peak list contributes to the multi-tool ground truth consensus)
- **MAVEN** (C++ detector with a machine-learning-based peak filtering method whose peak list contributes to the ground truth consensus)

## Examples

```
zz = sum((2*I(2:end-1)-I(1:end-2)-I(3:end)).^2)/(numel(I)*(max(I)-baseline)^2); % EIC global zigzag index in [0,4.0]
```

## Evaluation signals

- Global zigzag index values fall within the documented range [0, 4.0].
- Reproduce the Table 1 contrast: EIC_ID 167 (bad) scores low MCQ and high zigzag, while EIC_IDs 27/90/134/154 (high-background or single-peak but locally good) score low MCQ yet low zigzag.
- Verdict consistency: zigzag-based classification flags the known-bad EIC as bad without falsely rejecting the high-background or single-peak EICs that MCQ-only classification misjudges.
- Threshold sweep reproduces Recall, Precision, and F-Score trends when cutoffs are varied both individually and in combination against ground truth peaks detected by at least two of XCMS, MZmine, and MAVEN.
- Output schema check: tab-separated table with one row per EIC, metric-value columns, and a good/bad verdict column per metric.

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak — classifications based on it alone are biased for these cases.
- Setting an optimal cutoff threshold for each metric still is a challenge task; thresholds must be varied empirically.
- The LC/MS profile dataset and Matlab scripts are available only upon request and no license is specified, so the published pipeline cannot be re-run from a public repository.
- Machine-learning-based automatic peak identification (e.g., SVM) was not implemented; it would require compiling and curating a sufficient number of good and bad representative peaks for training that does not yet exist.

## Evidence

- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [intro] During data processing, the metric cutoff thresholds were varied, both individually and in combination,: "During data processing, the metric cutoff thresholds were varied, both individually and in combination,"
- [methods] 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
