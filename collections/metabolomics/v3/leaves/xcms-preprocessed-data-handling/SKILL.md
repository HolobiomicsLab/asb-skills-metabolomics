---
name: xcms-preprocessed-data-handling
description: Use when untargeted LC–MS data has been pre-processed with XCMS (peak-picking,
  RT correction, grouping) and must be turned into an M11 metric matrix (e.g. 500
  peaks x 11 metrics) for Pass/Fail peak-quality classification or evaluation.
when_to_use_negative:
- Input is vendor-processed output in a unique proprietary format — MetaClean requires
  an XCMS object input and is not amenable to uniquely formatted vendor-processed
  outputs.
- Input is raw LC–MS data still needing peak-picking, RT correction, and grouping
  — run the XCMS pre-processing step first before handling the preprocessed object.
- Peaks already carry Pass/Fail labels and only need evaluation — skip prediction
  and go directly to calculateEvaluationMeasures/MLmetrics.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3893
tools:
- name: XCMS
  role: Performs peak-picking, retention time correction, and grouping across samples
    and QCs to produce the XCMS object MetaClean consumes
- name: MetaClean
  role: Computes the 11 M11 peak-quality metrics and assigns Pass/Fail labels via
    getPredicitons (caret predict) using the pre-trained AdaBoost model
  repo: https://github.com/KelseyChetnik/MetaClean
- name: caret
  role: Hosts the AdaBoost.M1-trained model (nIter=150) and performs the predict calls
    behind getPredicitons
- name: fastAdaboost
  role: Trains the AdaBoost classifier used for the global peak-quality model
- name: MLmetrics
  role: Computes evaluation measures (Accuracy, Pass and Fail F-measures) against
    expert labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:2962ee4f8a598a7e1d673c9e6b31959ad334109d7f4c3712638e3d2fe6d01437
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/KelseyChetnik/MetaClean
  tools:
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3893
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# xcms-preprocessed-data-handling

## Summary

Skill for taking XCMS-preprocessed LC–MS data (peak-picked, RT-corrected, grouped features) and converting it into the peak-quality metric matrices that MetaClean's classifier consumes, then scoring the classifier's Pass/Fail predictions against expert labels.

## When to use

Use when untargeted LC–MS data has been pre-processed with XCMS (peak-picking, RT correction, grouping) and must be turned into an M11 metric matrix (e.g. 500 peaks x 11 metrics) for Pass/Fail peak-quality classification or evaluation.

## When NOT to use

- Input is vendor-processed output in a unique proprietary format — MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs.
- Input is raw LC–MS data still needing peak-picking, RT correction, and grouping — run the XCMS pre-processing step first before handling the preprocessed object.
- Peaks already carry Pass/Fail labels and only need evaluation — skip prediction and go directly to calculateEvaluationMeasures/MLmetrics.

## Inputs

- XCMS object (output of XCMS peak-picking, retention time correction, and grouping across samples and QCs)
- Pre-trained AdaBoost M11 classifier (nIter=150, trained on ST000726, caret package)
- Peak-quality metric matrix (n peaks x 11 M11 metrics)
- Expert Pass/Fail reference labels (Petrick) for evaluation

## Outputs

- Per-peak Pass/Fail quality predictions for the test dataset (e.g. 500 peaks from ST000695)
- results_table.csv containing per-peak predictions and evaluation measures (Accuracy, Pass F-measure, Fail F-measure)

## How to apply

Run XCMS peak-picking, retention time correction, and grouping across samples and QCs to produce an XCMS object, since MetaClean requires an XCMS object input and computes its 11 M11 peak-quality metrics (adapted from Zhang et al. 2014 and TargetedMSQC) from it. For reproducibility of Test 1 (ST000695), load the pre-trained AdaBoost M11 model trained on the full ST000726 development set with nIter=150 via AdaBoost.M1 in the caret package, and load the Test 1 metric matrix of 500 peaks x 11 M11 metrics. Use MetaClean::getPredicitons (caret predict) to assign Pass/Fail labels, binarizing probabilistic predictions at the balanced-accuracy-optimal probability threshold determined during fivefold CV on the development set. Evaluate Accuracy and Pass/Fail F-measures against the expert (Petrick) Pass/Fail reference labels using calculateEvaluationMeasures/MLmetrics, and save per-peak predictions and evaluation measures to results_table.csv. Performance must exceed the 100-permutation random-chance counterpart on the same test set.

## Related tools

- **XCMS** (Performs peak-picking, retention time correction, and grouping across samples and QCs to produce the XCMS object MetaClean consumes)
- **MetaClean** (Computes the 11 M11 peak-quality metrics and assigns Pass/Fail labels via getPredicitons (caret predict) using the pre-trained AdaBoost model) — https://github.com/KelseyChetnik/MetaClean
- **caret** (Hosts the AdaBoost.M1-trained model (nIter=150) and performs the predict calls behind getPredicitons)
- **fastAdaboost** (Trains the AdaBoost classifier used for the global peak-quality model)
- **MLmetrics** (Computes evaluation measures (Accuracy, Pass and Fail F-measures) against expert labels)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean"); library(MetaClean); preds <- getPredicitons(model = adaboost_M11_model, data = test1_metric_matrix)
```

## Evaluation signals

- Input must be an XCMS object; vendor-processed outputs are incompatible and should trigger a pre-processing step first.
- Metric matrix dimensionality check: Test 1 yields 500 peaks x 11 M11 metrics; a different dimensionality indicates pre-processing or metric computation errors.
- Predictions must be binarized at the balanced-accuracy-optimal probability threshold from CV on ST000726, not an arbitrary 0.5 cutoff.
- Expected Test 1 results: Accuracy = 0.81, F-measure Pass = 0.87, Fail = 0.62; classifier performance must exceed the 100-permutation random-chance counterpart on the same test set.
- results_table.csv must contain both per-peak predictions and the evaluation measures for downstream inspection.

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class (F = 0.62) than the high-quality one, partly attributed to the smaller number of low-quality peaks in the development sets.
- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs.
- Ionization mode, analytical platform and method, and biological matrix affect peak shape and size, so performance on datasets from a different platform (e.g. Test 2) may drop relative to same-platform tests like Test 1.

## Evidence
