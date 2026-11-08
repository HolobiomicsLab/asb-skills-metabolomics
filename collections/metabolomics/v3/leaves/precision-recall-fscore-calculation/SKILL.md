---
name: precision-recall-fscore-calculation
description: 'Use when evaluating chromatographic peak quality filtering: you have
  a detected peak list with quality metric values and a ground truth peak list, and
  need Recall/Precision/F-Score at varied metric cutoff thresholds.'
when_to_use_negative:
- When the goal is evaluating extracted ion chromatogram (EIC) quality rather than
  detected chromatographic peak quality — use the EIC-specific metrics (MCQ index,
  global zigzag index) instead.
- When no ground truth peak list exists (e.g. peaks cannot be cross-validated by at
  least two of XCMS, MZmine, MAVEN); Recall/Precision/F-Score cannot be computed without
  one.
- When you only need a single best metric identified by case-study evaluation rather
  than a comprehensive threshold sweep across individual and combined metrics.
tools:
- name: Matlab
  role: Host environment for the four-module data processing program that computes
    the peak quality metrics and performs the threshold sweep
- name: XCMS
  role: One of the three tools whose jointly detected peaks define the ground truth
    peak list used to score Recall/Precision/F-Score
- name: MZmine
  role: One of the three tools whose jointly detected peaks define the ground truth
    peak list used to score Recall/Precision/F-Score
- name: MAVEN
  role: One of the three tools whose jointly detected peaks define the ground truth
    peak list used to score Recall/Precision/F-Score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:b4103e11f2e13d9ed79ef114b936fde3f247e2edccce34b5e45bb6d3fdaf8594
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# precision-recall-fscore-calculation

## Summary

Systematically varies peak quality metric cutoff thresholds (individually and in combination) against a ground truth peak list to compute Recall, Precision, and F-Score for evaluating LC/MS chromatographic peak filtering performance, as described in Zhang & Zhao's Matlab four-module LC/MS processing program.

## When to use

Use when evaluating chromatographic peak quality filtering: you have a detected peak list with quality metric values and a ground truth peak list, and need Recall/Precision/F-Score at varied metric cutoff thresholds.

## When NOT to use

- When the goal is evaluating extracted ion chromatogram (EIC) quality rather than detected chromatographic peak quality — use the EIC-specific metrics (MCQ index, global zigzag index) instead.
- When no ground truth peak list exists (e.g. peaks cannot be cross-validated by at least two of XCMS, MZmine, MAVEN); Recall/Precision/F-Score cannot be computed without one.
- When you only need a single best metric identified by case-study evaluation rather than a comprehensive threshold sweep across individual and combined metrics.

## Inputs

- Detected chromatographic peak list with six quality metric values (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) computed by the Matlab four-module processing program
- Ground truth peak list defined as peaks detected by at least two of XCMS, MZmine, and MAVEN (matched with 0.05 Da m/z tolerance and 5 seconds retention time tolerance)
- Metric cutoff threshold ranges to sweep (individually and in combination)

## Outputs

- Recall, Precision, and F-Score values at each threshold setting, computed for the whole dataset
- F-Score versus threshold curves for the six peak quality metrics
- Combined-metric performance results (Recall, Precision, F-Score at joint thresholds)
- peak_metric_threshold_sweep_results.csv

## How to apply

Match each detected peak against the ground truth list (peaks detected by at least two of XCMS, MZmine, and MAVEN, using 0.05 Da m/z tolerance and 5 seconds retention time tolerance) to compute true positives per TP(X)=N(X∩TruePeaklist). Select one peak quality metric at a time (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index), systematically vary its cutoff threshold, and at each threshold compute Recall, Precision, and F-Score over the whole dataset. Then combine all six metrics at the reported joint thresholds — Sharpness_Th=2.0, Gaussian_Similarity_Th=0.6, SNR_Th=1.3, Peak_Significance_Th=1.2, TPASR_Th=0.8, Zigzag index=0.9 — and recompute the same three statistics. Write all per-metric and combined results to a CSV and plot F-Score versus threshold curves for the six metrics. This comprehensive threshold-sweep approach is grounded in the paper's four-module Matlab processing program, where module 4 is peak quality evaluation and filtering.

## Related tools

- **Matlab** (Host environment for the four-module data processing program that computes the peak quality metrics and performs the threshold sweep)
- **XCMS** (One of the three tools whose jointly detected peaks define the ground truth peak list used to score Recall/Precision/F-Score)
- **MZmine** (One of the three tools whose jointly detected peaks define the ground truth peak list used to score Recall/Precision/F-Score)
- **MAVEN** (One of the three tools whose jointly detected peaks define the ground truth peak list used to score Recall/Precision/F-Score)

## Evaluation signals

- Combined-metric run must yield Recall = 0.7076, Precision = 0.6186, and F-Score = 0.6601
- Maximum single-metric F-Score must remain smaller than 0.6, confirming combined metrics outperform individual metrics
- F-Score curve maxima must occur for sharpness, peak significance level, and zigzag index, matching the paper's reported ranking
- All per-metric and combined performance results must be written to peak_metric_threshold_sweep_results.csv
- True positives must be computed strictly as TP(X)=N(X∩TruePeaklist) with the 0.05 Da m/z and 5 s retention time matching tolerances

## Limitations

- Ground truth peaks are defined by consensus of only three tools (XCMS, MZmine, MAVEN), so peaks missed by all three are invisible to the evaluation.
- Optimal cutoff thresholds for each quality metric remain an unsolved challenge — the swept thresholds are empirical, not provably optimal.
- Machine learning-based automatic peak identification (e.g., SVM) is not yet implemented because a sufficient curated training set of 'good' and 'bad' representative peaks does not yet exist.

## Evidence

- [intro] During data processing, the metric cutoff thresholds were varied, both individually and in combination,: "During data processing, the metric cutoff thresholds were varied, both individually and in combination,"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [abstract] Using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
