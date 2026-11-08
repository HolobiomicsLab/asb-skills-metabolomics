---
name: svm-classifier-training
description: Use when a curated, expert-labeled set of representative good and bad
  chromatographic peaks exists and you want a single combined classifier for peak
  quality filtering instead of varying individual metric cutoff thresholds.
when_to_use_negative:
- No curated good/bad training peak set exists — the article states machine learning-based
  peak identification is not yet implemented because the required training samples
  have not been compiled; fall back to individual metric cutoff-threshold filtering.
- The goal is to evaluate extracted EIC quality rather than detected peak quality
  — use the EIC-related global zigzag index / MCQ index evaluation path instead.
- The goal is to characterize individual metrics' cutoff-threshold behavior (Recall/Precision/F-score
  sweeps, individual and combined), not to build a supervised classifier.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: Matlab
  role: Implementation language of the authors' four-module data processing program;
    would compute the six peak quality metrics, train the SVM, run cross-validation,
    and write the performance table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:dc83eaa53e2a318d62bcc563e8cab303626223dd873c507e583fadc3dd3e121e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# svm-classifier-training

## Summary

Trains a support vector machine to classify LC/MS chromatographic peaks as good or bad from six normalized quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index), implementing the paper's proposed but unrealized alternative to per-metric cutoff thresholds.

## When to use

Use when a curated, expert-labeled set of representative good and bad chromatographic peaks exists and you want a single combined classifier for peak quality filtering instead of varying individual metric cutoff thresholds.

## When NOT to use

- No curated good/bad training peak set exists — the article states machine learning-based peak identification is not yet implemented because the required training samples have not been compiled; fall back to individual metric cutoff-threshold filtering.
- The goal is to evaluate extracted EIC quality rather than detected peak quality — use the EIC-related global zigzag index / MCQ index evaluation path instead.
- The goal is to characterize individual metrics' cutoff-threshold behavior (Recall/Precision/F-score sweeps, individual and combined), not to build a supervised classifier.

## Inputs

- Curated training set of expert-labeled representative good and bad chromatographic peaks from LC/MS metabolomics data
- Per-peak feature vectors of the six quality metrics: sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index
- LC/MS profile dataset processed by the four-module pipeline (acute EIC extraction → EIC quality evaluation/filtering → chromatographic peak detection → peak quality evaluation/filtering)

## Outputs

- Trained SVM classifier mapping normalized six-metric feature vectors to good/bad peak classes
- Table file of cross-validated classification performance (accuracy, recall, precision, F-score)

## How to apply

Compile a curated training set of representative 'good' and 'bad' peaks (the article notes this set does not yet exist and must be built for training). For each peak compute the six quality metrics — sharpness, Gaussian similarity, SNR, peak significance level, TPASR (triangle peak area similarity ratio), and the local peak-related zigzag index — and normalize each metric feature to a common scale (e.g., subtract the mean and divide by the maximum value) so no single metric dominates the classifier. Train the SVM, e.g. in the authors' Matlab data processing program, using the six normalized metrics as input features and the expert-assigned good/bad labels as the class variable. Cross-validate the trained SVM and report accuracy plus comparable metrics such as recall, precision, and F-score for distinguishing good from bad peaks. Save the cross-validated performance results to a table file; the combined-metric design is justified by the paper's finding that using several metrics in combination is more efficient than any individual metric.

## Related tools

- **Matlab** (Implementation language of the authors' four-module data processing program; would compute the six peak quality metrics, train the SVM, run cross-validation, and write the performance table)

## Evaluation signals

- Cross-validated accuracy/recall/precision/F-score on held-out folds shows the combined-metric SVM automatically distinguishes good from bad peaks, consistent with the finding that combined metrics are more efficient than individual ones.
- Feature matrix has exactly six columns (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index), each normalized to a common scale before training.
- Class variable values match expert-assigned good/bad labels for every training peak; no unlabeled peaks enter training.
- Cross-validated performance values are reported numerically and saved to a table file.

## Limitations

- The SVM classifier is a proposed extension, not an implemented method: the article states it is not yet implemented and requires a curated training set of good and bad representative peaks that does not yet exist.
- The LC/MS profile dataset and Matlab scripts are available only upon request (no public repository) and no software license is specified, so training data must be obtained or recreated.
- Performance depends on the quality of expert-assigned good/bad labels, which introduces curation subjectivity.
- Optimal cutoff thresholds per metric remain an unsolved challenge in the paper; the SVM sidesteps threshold-setting but inherits the label-quality problem.

## Evidence

- [intro] We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio: "We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio"
- [abstract] Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [discussion] This requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples.: "This requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples."
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
