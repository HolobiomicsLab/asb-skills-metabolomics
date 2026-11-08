---
name: chromatographic-peak-quality-assessment
description: Use when extracted ion chromatograms (EICs) from LC/MS metabolomics profiles
  have already been peak-detected and individual peaks (or EICs) must be scored, filtered,
  or classified as good vs. bad before downstream processing.
when_to_use_negative:
- No labeled set of curated 'good' and 'bad' representative peaks exists and ground
  truth peaks cannot be assembled from multiple tools — the paper notes SVM-based
  automatic peak identification 'requires compiling and curating a sufficient number
  of "good" and "bad" representative peaks for training samples', which does not yet
  exist.
- The input is raw LC/MS profile data that has not yet passed acute EIC extraction
  and chromatographic peak detection — quality evaluation is a later module, not a
  substitute for peak detection.
- EIC-level (not peak-level) quality scoring is the goal for high-background or single-peak
  EICs — the MCQ index is unsuited there, so use the EIC-related global zigzag index
  instead of peak-level metrics.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Matlab
  role: Home of the paper's four-module data processing program that computes the
    six peak quality metrics, varies cutoff thresholds, and trains/cross-validates
    the SVM good/bad peak classifier
- name: XCMS
  role: R-based peak detection tool; one of the three tools whose detected peaks (matched
    by ≥2 tools) define ground truth peaks for evaluating metric performance
- name: MZmine
  role: Java-based peak detection tool ('centroidPicker'); contributes peak lists
    for ground truth peak definition
- name: MAVEN
  role: C++ tool with a complex, machine learning-based peak filtering method; contributes
    peak lists for ground truth peak definition
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:0907845aa79ce4913bb09ef3e95b76a7db43fde99ee22906e290e190b0648551
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chromatographic-peak-quality-assessment

## Summary

Evaluates the quality of chromatographic peaks in LC/MS-based metabolomics data by computing six peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag index) and combining them — via individually varied cutoff thresholds or an SVM classifier — to filter out 'bad' peaks more efficiently than any single metric.

## When to use

Use when extracted ion chromatograms (EICs) from LC/MS metabolomics profiles have already been peak-detected and individual peaks (or EICs) must be scored, filtered, or classified as good vs. bad before downstream processing.

## When NOT to use

- No labeled set of curated 'good' and 'bad' representative peaks exists and ground truth peaks cannot be assembled from multiple tools — the paper notes SVM-based automatic peak identification 'requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples', which does not yet exist.
- The input is raw LC/MS profile data that has not yet passed acute EIC extraction and chromatographic peak detection — quality evaluation is a later module, not a substitute for peak detection.
- EIC-level (not peak-level) quality scoring is the goal for high-background or single-peak EICs — the MCQ index is unsuited there, so use the EIC-related global zigzag index instead of peak-level metrics.

## Inputs

- Extracted ion chromatograms (EICs) extracted from an LC/MS profile dataset (e.g., generated on an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer)
- Detected chromatographic peak lists from peak detection tools (XCMS 'MatchedFilter'/'centWave', MZmine 'centroidPicker', MAVEN ML-based filtering)
- A labeled training set of expert-curated representative good and bad peaks (required for the SVM variant)
- Ground truth peak lists from at least two of XCMS, MZmine, and MAVEN (for threshold-based performance evaluation)

## Outputs

- Per-peak/per-EIC values for the six quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) and EIC-level global zigzag index
- Filtered peak/EIC lists after applying metric cutoff thresholds individually and in combination
- Cross-validated classification performance table (accuracy, recall, precision, F-score) for the SVM good/bad peak classifier

## How to apply

In the paper's four-module Matlab program, this skill occupies modules 2 and 4 (EIC quality evaluation and filtering, then peak quality evaluation and filtering). Compute the six metrics for each detected peak — sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio (TPASR), and local peak-related zigzag index — noting that sharpness, peak significance level, and zigzag index perform best due to the zigzag nature of LC/MS chromatographic peaks. Either vary each metric's cutoff threshold individually and in combination while scoring Recall, Precision, and F-Score against ground truth peaks (peaks detected by at least two of XCMS, MZmine, and MAVEN), or train an SVM: normalize each metric feature to a common scale (subtract the mean, divide by the maximum), train on expert-assigned good/bad labels, and report cross-validated accuracy/recall/precision/F-score. For EICs that the mass chromatographic quality (MCQ) index cannot fairly evaluate — those with high background signals or only a single peak — use the proposed EIC-related global zigzag index based on first-order derivatives.

## Related tools

- **Matlab** (Home of the paper's four-module data processing program that computes the six peak quality metrics, varies cutoff thresholds, and trains/cross-validates the SVM good/bad peak classifier)
- **XCMS** (R-based peak detection tool; one of the three tools whose detected peaks (matched by ≥2 tools) define ground truth peaks for evaluating metric performance)
- **MZmine** (Java-based peak detection tool ('centroidPicker'); contributes peak lists for ground truth peak definition)
- **MAVEN** (C++ tool with a complex, machine learning-based peak filtering method; contributes peak lists for ground truth peak definition)

## Examples

```
% MATLAB, per the paper's Matlab program: featNorm = bsxfun(@rdivide, feats - mean(feats), max(feats)); svmstruct = svmtrain(featNorm, labels); [pred, acc] = svmclassify(svmstruct, featNormCV)
```

## Evaluation signals

- Cross-validated SVM performance (accuracy, recall, precision, F-score) is saved to a table file and demonstrates that the six-metric combination automatically distinguishes good from bad peaks.
- Threshold-based evaluation: Recall, Precision, and F-Score are reported as cutoffs are varied individually and in combination, and the combined-metric performance should exceed any single metric's performance.
- Ground truth check: peaks counted as correct detections must be found by at least two of XCMS, MZmine, and MAVEN on the same UPLC/MS dataset.
- Feature normalization check: each of the six metric features is mean-subtracted and divided by its maximum so no single metric dominates the SVM.
- Robustness check: EICs with high background signals or only a single peak — cases where the MCQ index fails — must be scored successfully by the global zigzag index.

## Limitations

- Setting an optimal cutoff threshold for each metric 'still is a challenge task' — thresholds are data-dependent and the paper does not prescribe fixed values.
- The LC/MS profile dataset and Matlab scripts are 'available upon request' with no public repository, and no license is specified, limiting reproducibility.
- The MCQ index cannot fairly evaluate EICs with high background signals or EICs containing only a single peak, so peak/EIC metrics must be chosen per scenario.
- The ML/SVM route requires a curated training set of representative good and bad peaks that the paper identifies as not yet existing.

## Evidence

- [abstract] six peak quality metrics cover all proposed cutoffs: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] combined metrics outperform individual ones: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] four-module pipeline placement: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [methods] ground truth peak definition: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] SVM training-set prerequisite: "This requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples."
- [discussion] cutoff threshold challenge: "setting an optimal cutoff threshold for each metric still is a challenge task"
