---
name: lcms-peak-quality-feature-engineering
description: Use when detected chromatographic peaks from LC/MS metabolomics data
  must be classified as good or bad, and single-metric cutoff filtering is insufficient
  — combine multiple peak quality metrics as ML features.
when_to_use_negative:
- When evaluating extracted ion chromatograms (EICs) rather than detected peaks —
  the paper shows EICs with high background signals or containing only a single peak
  cannot be fairly evaluated by the MCQ index and instead need the EIC-related global
  zigzag index
- When no curated training set of good and bad representative peaks exists — the article
  explicitly notes machine-learning-based peak identification via SVM is not yet implemented
  because such training samples are lacking
- When a single metric with an optimized cutoff threshold suffices for the filtering
  task and interpretability of a simple threshold rule is preferred
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Matlab
  role: Custom data processing program used to compute the six peak quality metrics
    and train/cross-validate the SVM classifier on labeled good/bad peaks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:cb50380a00076489a02cf135b566e38d26c078234b192406aa73aac44b00cf8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# lcms-peak-quality-feature-engineering

## Summary

Engineer chromatographic peak-quality feature vectors for LC/MS metabolomics peaks by computing the six quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag index) in Matlab, then normalize and combine them for SVM-based good/bad peak classification instead of applying single-metric cutoff thresholds.

## When to use

Use when detected chromatographic peaks from LC/MS metabolomics data must be classified as good or bad, and single-metric cutoff filtering is insufficient — combine multiple peak quality metrics as ML features.

## When NOT to use

- When evaluating extracted ion chromatograms (EICs) rather than detected peaks — the paper shows EICs with high background signals or containing only a single peak cannot be fairly evaluated by the MCQ index and instead need the EIC-related global zigzag index
- When no curated training set of good and bad representative peaks exists — the article explicitly notes machine-learning-based peak identification via SVM is not yet implemented because such training samples are lacking
- When a single metric with an optimized cutoff threshold suffices for the filtering task and interpretability of a simple threshold rule is preferred

## Inputs

- Labeled training set of curated representative good and bad chromatographic peaks (peak list from LC/MS profile data)
- Raw extracted-ion-chromatogram peak profiles from LC/MS-based metabolomics data (e.g. Agilent HPLC / Q-TOF Premier profile dataset) for computing the six metrics
- Expert-assigned good/bad class labels for each training peak

## Outputs

- Per-peak feature table of the six normalized quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index)
- Cross-validated SVM classification performance table (accuracy, recall, precision, F-score for good vs. bad peaks)

## How to apply

Start from a labeled training set of curated representative good and bad chromatographic peaks detected in the LC/MS profile data (e.g. from an Agilent HPLC–Q-TOF Premier acquisition), and compute the six quality metrics — sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio (TPASR), and local zigzag index — for each peak to form its feature vector. Normalize each metric to a common scale (subtract the mean and divide by the maximum value) so no single metric dominates the classifier. Train a support vector machine (SVM) in Matlab on the labeled feature vectors, using the six metrics as inputs and the expert-assigned good/bad labels as the class variable. Perform cross-validation and report accuracy plus Recall, Precision, and F-Score for distinguishing good from bad peaks. The rationale is the paper's finding that using several peak quality metrics in combination is more efficient than any individual metric, and that sharpness, peak significance level, and zigzag index are the strongest individual discriminators due to the zigzag nature of LC/MS chromatographic peaks.

## Related tools

- **Matlab** (Custom data processing program used to compute the six peak quality metrics and train/cross-validate the SVM classifier on labeled good/bad peaks)

## Evaluation signals

- Feature table contains exactly the six paper-defined metrics per peak, each normalized (mean-subtracted, divided by the maximum) so no metric dominates
- SVM cross-validated accuracy (plus recall, precision, F-score) is reported and demonstrates automatic separation of good from bad peaks
- Combined-metric classification performance exceeds the performance achievable by any individual metric cutoff threshold applied alone
- Ground truth for evaluation agrees with the paper's convention: a peak is considered a ground truth peak if it can be detected by multiple tools (at least two of XCMS, MZmine, MAVEN)
- Cross-validated performance results are written to a table file as the final deliverable

## Limitations

- Compiling and curating a sufficient number of good and bad representative peaks for training samples is required but does not yet exist; machine learning-based automatic peak identification (e.g. SVM) was not yet implemented in the paper
- Setting an optimal cutoff threshold for each quality metric still is a challenge task
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or license for usage is provided
- Training labels depend on expert-assigned good/bad judgments (or multi-tool ground truth), so classifier quality is bounded by curation effort

## Evidence

- [abstract] Of the six peak quality evaluation metrics, sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Combining metrics beats individual metrics.: "using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] Matlab program modules.: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [intro] Threshold variation in evaluation.: "During data processing, the metric cutoff thresholds were varied, both individually and in combination,"
- [methods] Ground truth definition.: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] Missing training set limitation.: "This requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples."
