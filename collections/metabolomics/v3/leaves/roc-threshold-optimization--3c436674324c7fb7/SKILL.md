---
name: roc-threshold-optimization
description: Use when a trained binary classifier outputs probability scores for LC-MS
  peak groups and you need to characterize discriminative performance and pick an
  operating point.
when_to_use_negative:
- The classifier has not yet produced probability scores for each peak group
- Peak group labels are not manually reviewed so no ground-truth true/false peaks
  exist to compute rates
- A single fixed threshold has already been chosen and no threshold sweep is needed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: LC-MS
  role: Source instrument producing the nontargeted HILIC-based spectral data whose
    peaks are classified and scored before threshold evaluation
- name: R
  role: Custom R scripts environment in which ROC curve computation and metric reporting
    are implemented
- name: MZmine 2
  role: Peak extraction upstream that generates the peak lists whose peak groups receive
    DNN probability scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:d233d4ba602b1ea9f1b83ec872030b7b160fc0858aefb2df866983a74b3a7904
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LC-MS
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# roc-threshold-optimization

## Summary

Computes receiver operating characteristic curves from the trained LC-MS peak-classification model's probability scores and selects the optimum score threshold, reporting sensitivity (true positive rate) and specificity (false positive rate) at each cutoff.

## When to use

Use when a trained binary classifier outputs probability scores for LC-MS peak groups and you need to characterize discriminative performance and pick an operating point.

## When NOT to use

- The classifier has not yet produced probability scores for each peak group
- Peak group labels are not manually reviewed so no ground-truth true/false peaks exist to compute rates
- A single fixed threshold has already been chosen and no threshold sweep is needed

## Inputs

- Probability scores assigned by the trained DNN to each LC-MS peak group
- Manually reviewed peak labels (true vs false) for training/calibration/test sets: 1304 training, 740 calibration, 726 test

## Outputs

- Receiver operating characteristic curve with AUC (expected ~0.992 training, ~0.988 test)
- Optimum operating point score threshold (0.4725) with corresponding true positive rate (~0.945) and false positive rate (~0.04)
- dnn_peak_classification_metrics.csv

## How to apply

After the image-based deep neural network assigns each peak group a probability score of being a true peak, evaluate the score at every threshold cutoff. At each cutoff, determine the true positive rate (sensitivity) and true negative rate (specificity) to construct the receiver operating characteristic curve. The article reports AUC values of 0.992 (training) and 0.988 (test) on Cohort 1 peaks (1304 training / 740 calibration / 726 test manually reviewed examples). Select the optimum operating point score threshold of 0.4725, which corresponds to a true positive rate of approximately 0.945 and a false positive rate of approximately 0.04; the article notes selection criteria are provided in the Supporting Information. Write the threshold and rates to a metrics file and render the ROC curve.

## Related tools

- **LC-MS** (Source instrument producing the nontargeted HILIC-based spectral data whose peaks are classified and scored before threshold evaluation)
- **R** (Custom R scripts environment in which ROC curve computation and metric reporting are implemented)
- **MZmine 2** (Peak extraction upstream that generates the peak lists whose peak groups receive DNN probability scores)

## Examples

```
python EDML_deep_learning2.py --datasets https://doi.org/doi:10.25345/C5FD2F --output dnn_peak_classification_metrics.csv
```

## Evaluation signals

- Training-set AUC should be approximately 0.992 and test-set AUC approximately 0.988
- Score threshold of 0.4725 should yield a true positive rate of approximately 0.945
- The same threshold should yield a false positive rate of approximately 0.04
- ROC curve must be computed across all score threshold cutoff values, not a single point
- Class split at training should be 50% true peaks and 50% false peaks of 1304 manually reviewed examples

## Limitations

- Selection criteria for the 0.4725 optimum operating point are only described in the Supporting Information, not in the main text
- Performance metrics (AUC, TPR, FPR) are reported for Cohort 1 peaks; behavior on other cohorts (e.g. Cohort 2) is not given in these results
- The 50/50 true/false peak split in training is a balanced design; threshold behavior on unbalanced data is not characterized here

## Evidence
