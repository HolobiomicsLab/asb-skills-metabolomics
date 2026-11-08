---
name: untargeted-lcms-feature-labeling
description: Use when an untargeted LC-MS feature table (XCMS output) contains poorly
  integrated peaks that manual quality assessment would otherwise need to catch, and
  you want automated Pass/Fail flagging before downstream statistics.
when_to_use_negative:
- Input is vendor-processed output rather than an XCMS object, since MetaClean in
  its current form does not accept uniquely formatted vendor-processed results.
- The dataset contains few examples of low-abundance Pass peaks — the source study
  reports the classifier is conservative and performs worse on the Fail class when
  the development set lacks such examples.
- You only need reproducibility filtering on pooled QC samples rather than integration-quality
  assessment; a simple RSD < 30% cutoff on pooled QCs may suffice and is the alternative
  baseline the paper compares against.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaClean
  role: R package that computes the 12 peak-quality metrics and provides the trained
    classifiers to flag low-quality peaks in untargeted LC-MS data
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Upstream preprocessing that performs peak-picking, retention time correction,
    and grouping of peaks across samples and QCs to produce the feature table MetaClean
    consumes
- name: caret
  role: Provides the eight classification algorithms used to build the candidate Pass/Fail
    classifiers from the peak-quality metric sets
- name: scmamp
  role: R package used to perform Friedman's test and Nemenyi's post-hoc correction
    when statistically comparing candidate classifiers
- name: MLmetrics
  role: Used to compute the classification evaluation measures (accuracy, balanced
    accuracy, sensitivity, specificity, PPV, NPV, F-measure) for each candidate classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:8cc16de4a9018f7a873b53c0395b9ed042cfb29c46935d1dc369dc31ae419a86
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - scmamp
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# untargeted-lcms-feature-labeling

## Summary

Labels each XCMS-derived LC-MS feature in an untargeted metabolomics run as high-quality (Pass) or low-quality (Fail) using peak-quality metrics from Zhang et al. and TargetedMSQC paired with machine learning classifiers, as implemented in the MetaClean R package.

## When to use

Use when an untargeted LC-MS feature table (XCMS output) contains poorly integrated peaks that manual quality assessment would otherwise need to catch, and you want automated Pass/Fail flagging before downstream statistics.

## When NOT to use

- Input is vendor-processed output rather than an XCMS object, since MetaClean in its current form does not accept uniquely formatted vendor-processed results.
- The dataset contains few examples of low-abundance Pass peaks — the source study reports the classifier is conservative and performs worse on the Fail class when the development set lacks such examples.
- You only need reproducibility filtering on pooled QC samples rather than integration-quality assessment; a simple RSD < 30% cutoff on pooled QCs may suffice and is the alternative baseline the paper compares against.

## Inputs

- XCMS object (xcmsSet) from peak-picking, retention time correction, and grouping of an untargeted LC-MS metabolomics run
- Expert-labeled Pass/Fail training labels for a subset of peaks (e.g. 500 manually assessed peaks)
- Peak-quality metric matrices computed per peak (M4 = Zhang et al. metrics, M7 = TargetedMSQC-derived metrics, M11 = union of both)

## Outputs

- Per-peak Pass/Fail classification label for every feature in the LC-MS run
- Classifier performance statistics per candidate model: accuracy, balanced accuracy, sensitivity (recallPass), specificity (recallFail), PPV (precisionPass), NPV (precisionFail), and F-measures for Pass and Fail classes
- Friedman-Nemenyi statistical comparison of candidate classifiers identifying the global best-performing classifier

## How to apply

Run peak-picking, retention time correction, and grouping with XCMS to produce a feature table, then compute the MetaClean peak-quality metrics on each peak's extracted ion chromatogram: the Zhang et al. set (e.g. sharpness, Gaussian similarity, TPASR, local zigzag index) and the TargetedMSQC-derived set (e.g. apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency). Pair these metric sets (M4, M7, and their union M11) with candidate classifiers built in caret — Decision Tree, Logistic Regression, Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network, and Random Forest — and train on expert-labeled Pass/Fail peaks. During cross-validation, binarize predictions at the probability threshold that maximizes balanced accuracy in each fold. Apply Friedman's test followed by Nemenyi's post-hoc test (scmamp) to per-fold performance statistics to select the global classifier; in the source study AdaBoost with the M11 metric set was best. Finally, apply the selected classifier to flag each unlabeled feature as Pass or Fail, optionally comparing against RSD < 30% QC filtering.

## Related tools

- **MetaClean** (R package that computes the 12 peak-quality metrics and provides the trained classifiers to flag low-quality peaks in untargeted LC-MS data) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Upstream preprocessing that performs peak-picking, retention time correction, and grouping of peaks across samples and QCs to produce the feature table MetaClean consumes)
- **caret** (Provides the eight classification algorithms used to build the candidate Pass/Fail classifiers from the peak-quality metric sets)
- **scmamp** (R package used to perform Friedman's test and Nemenyi's post-hoc correction when statistically comparing candidate classifiers)
- **MLmetrics** (Used to compute the classification evaluation measures (accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, F-measure) for each candidate classifier)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean"); metaCleanPipeline(xcmsObj = xset, trainLabels = labels, classifier = "AdaBoost", metricSet = "M11")
```

## Evaluation signals

- The selected global classifier should be identified via two rounds of Friedman's and Nemenyi's tests — an intra-metric-set round followed by an inter-metric-set round — and should match AdaBoost with the M11 metric set (p = 0.001–0.005 for Pass, p = 0.0023 for Fail in the source study).
- Cross-validation predictions must be binarized at the probability threshold that maximizes balanced accuracy in each fold, not at a fixed 0.5 cutoff.
- Performance must be reported as the full set of seven measures (accuracy, balanced accuracy, recallPass, recallFail, precisionPass, precisionFail, F-measures for Pass and Fail) for all 24 candidate classifiers (3 metric sets × 8 algorithms).
- Results should be validated against the paper's observation that MetaClean outperformed the RSD < 30% QC-filtering baseline.
- The classifier's predictions on the development dataset should reproduce the observed class imbalance behavior — better performance on Pass than Fail — consistent with the smaller number of low-quality peaks in the training set.

## Limitations

- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs.
- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one for Test 1 and 2, partly because the development dataset contained fewer low-quality peaks.
- Many misclassified low-quality peaks in Test 2 were low abundance and difficult to distinguish from background noise; the development dataset did not contain many examples of such low-abundance Pass peaks.
- Ionization mode, analytical platform/method, and biological matrix affect peak shape and size, so the methodology must be considered in light of the characteristics of the specific LC-MS run.

## Evidence
