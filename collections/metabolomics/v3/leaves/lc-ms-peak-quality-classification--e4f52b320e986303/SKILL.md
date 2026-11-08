---
name: lc-ms-peak-quality-classification
description: Use when an untargeted LC-MS metabolomics feature table produced by XCMS
  contains poorly integrated peaks that would propagate to downstream analyses, and
  automated Pass/Fail peak-quality classification is needed instead of manual expert
  assessment.
when_to_use_negative:
- Input is not an XCMS object — MetaClean requires an XCMS object input and is not
  amenable to uniquely formatted vendor-processed outputs
- The goal is reproducibility filtering rather than integration-quality assessment
  — RSD < 30% filtering on pooled QC samples is the alternative traditional approach
  and may suffice
- 'The dataset contains many low-abundance peaks: the development dataset lacked low-abundance
  Pass examples, so the classifier misclassified low-abundance low-quality peaks indistinguishable
  from background noise'
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaClean
  role: R package implementing the peak-quality metrics and best-performing classifiers
    for flagging low-quality peaks in XCMS-based untargeted metabolomics pipelines
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Pre-processing of raw LC-MS data (peak-picking, retention time correction,
    grouping) to produce the feature data the classifier consumes
- name: caret
  role: R package used to train the candidate classification algorithms (e.g., AdaBoost)
    and the randomized counterpart classifiers
- name: IPO
  role: Optimizes XCMS parameters using isotope data prior to peak-quality classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:1a54c2a0dfd8af6f341daf902fb26997b223b8c502da842f81c90ee29c833574
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - caret
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-ms-peak-quality-classification

## Summary

Trains and applies machine-learning classifiers (e.g., AdaBoost over the M11 peak-quality metric set from the MetaClean framework) to distinguish high-quality (Pass) from low-quality (Fail) peak integrations in untargeted LC-MS metabolomics, replacing subjective manual curation and RSD-only filtering.

## When to use

Use when an untargeted LC-MS metabolomics feature table produced by XCMS contains poorly integrated peaks that would propagate to downstream analyses, and automated Pass/Fail peak-quality classification is needed instead of manual expert assessment.

## When NOT to use

- Input is not an XCMS object — MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs
- The goal is reproducibility filtering rather than integration-quality assessment — RSD < 30% filtering on pooled QC samples is the alternative traditional approach and may suffice
- The dataset contains many low-abundance peaks: the development dataset lacked low-abundance Pass examples, so the classifier misclassified low-abundance low-quality peaks indistinguishable from background noise

## Inputs

- XCMS object (peak-picked, RT-corrected, grouped untargeted LC-MS feature data) from raw metabolomics runs
- Expert-assigned Pass/Fail labels for development-set peaks (e.g., 500 peaks from MW ST000726)
- Peak-quality metric values (12 metrics adapted from Zhang et al. 2014 and TargetedMSQC)

## Outputs

- Trained peak-quality classifier (best performer: AdaBoost with the M11 metric set)
- Per-peak Pass/Fail predictions flagging low-quality integrations
- Performance measures on independent test sets: Accuracy, Pass F-measure, Fail F-measure

## How to apply

Pre-process raw LC-MS data with XCMS (peak-picking, retention time correction, grouping of peaks across samples and QCs) to obtain an XCMS object, then compute peak-quality metrics — the 12 MetaClean metrics adapted from Zhang et al. (2014) and TargetedMSQC (Eshghi et al. 2018) form the M11 set. Expert-label a development set of peaks (e.g., 500 peaks from Metabolomics Workbench ST000726) as Pass/Fail, then train classifiers pairing three metric sets (M4, M7, M11) with eight ML algorithms in the caret package in R, using fivefold cross-validation and Friedman's test followed by Nemenyi's test to select the best performer — AdaBoost with the M11 metric set won (Friedman-Nemenyi p = 0.001–0.005 for Pass F-measure). Validate the chosen global classifier on independent test sets (e.g., ST000695, MTBLS354, and two splits of MTBLS306 in each ionization mode) using Accuracy and Pass/Fail F-measures, optionally comparing against RSD < 30% filtering on pooled QCs and against 100 label-permuted random-chance counterparts. The framework is released as the MetaClean R package for pipeline integration.

## Related tools

- **MetaClean** (R package implementing the peak-quality metrics and best-performing classifiers for flagging low-quality peaks in XCMS-based untargeted metabolomics pipelines) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processing of raw LC-MS data (peak-picking, retention time correction, grouping) to produce the feature data the classifier consumes)
- **caret** (R package used to train the candidate classification algorithms (e.g., AdaBoost) and the randomized counterpart classifiers)
- **IPO** (Optimizes XCMS parameters using isotope data prior to peak-quality classification)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean")
```

## Evaluation signals

- AdaBoost with the M11 metric set must show significantly better F-measure for both Pass and Fail classes than the other 23 candidate classifiers (Friedman-Nemenyi p = 0.001–0.005)
- Real classifier's Accuracy and Pass/Fail F-measures on each independent test set must exceed the averaged measures of 100 label-permuted random-chance counterparts
- Classifier performance on Test 1–4 should be compared against RSD < 30% filtering of peaks in pooled QC samples as a baseline
- Classifier should perform better on the Pass (high-quality) class than the Fail class for Test 1 and Test 2, reflecting its conservative behavior
- Verify the trained classifier is reproducible from the released MetaClean R package and that metrics match the 12 published peak-quality metrics

## Limitations

- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs
- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one for Test 1 and 2, partly attributed to the smaller number of low-quality peaks in the classifier development sets
- Many misclassified low-quality peaks in Test 2 were low abundance and difficult to distinguish from background noise; the development dataset did not contain many low-abundance Pass peak examples
- Ionization mode, analytical platform and method, and biological matrix affect peak shape and size, and thus integration quality, and must be considered when applying the methodology
- No exact performance metric values were reported for Test 1 (Accuracy and Pass/Fail F-measures described qualitatively)

## Evidence
