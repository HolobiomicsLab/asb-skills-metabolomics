---
name: sensitivity-specificity-rate-computation
description: Use when you have manually labeled true/false LC-MS peak groups and a
  classifier that assigns each group a probability score, and you need ROC-based sensitivity/specificity
  rates or retention/elimination fractions at specific thresholds.
when_to_use_negative:
- Peak groups are unlabeled — sensitivity/specificity cannot be computed without manual
  true/false classification
- The classifier outputs hard labels rather than continuous probability scores, preventing
  threshold-sweep ROC analysis
- The goal is raw feature extraction from LC-MS data rather than evaluating a peak-classification
  model against labels
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Custom R scripts used to compute rates, ROC curves, and summary tables from
    peak group classifications
- name: EDML_deep_learning2.py
  role: Script used to train the deep neural network model whose score thresholds
    are evaluated for sensitivity and specificity
  repo: JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: MZmine 2
  role: Generates the candidate peak groups (more/less restrictive settings) that
    form the labeled evaluation set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:c98addb9da5b8545314c0408944e0ff38303ffee36ef6bfc70251fe603a31e62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sensitivity-specificity-rate-computation

## Summary

Compute true positive rate (sensitivity), true negative rate (specificity), true-peak retention, and false-peak elimination rates for a binary peak classifier (e.g., the image-based DNN scoring LC-MS peak groups) at each score threshold, and select an optimum operating point such as the 0.4725 cutoff used in this study.

## When to use

Use when you have manually labeled true/false LC-MS peak groups and a classifier that assigns each group a probability score, and you need ROC-based sensitivity/specificity rates or retention/elimination fractions at specific thresholds.

## When NOT to use

- Peak groups are unlabeled — sensitivity/specificity cannot be computed without manual true/false classification
- The classifier outputs hard labels rather than continuous probability scores, preventing threshold-sweep ROC analysis
- The goal is raw feature extraction from LC-MS data rather than evaluating a peak-classification model against labels

## Inputs

- Manually classified peak groups with true/false labels (e.g., the 3000 Cohort 2 peak groups)
- Per-peak-group probability scores from the image-based deep neural network
- Per-peak-group classifications from the 15-parameter multiple logistic regression model
- Conventional baseline feature counts (e.g., 11,026 features from more restrictive MZmine 2 settings)

## Outputs

- Receiver operating characteristic (ROC) curves with AUC (e.g., 0.992 training / 0.988 test)
- Optimum score threshold with corresponding sensitivity and specificity (e.g., 0.4725; TPR ~0.945, FPR ~0.04)
- Per-model true-peak retention and false-peak elimination rates (e.g., 88% retained / 89% eliminated)
- Summary metrics table (dnn_param_filter_metrics.csv)

## How to apply

Score each manually classified peak group with the model (e.g., the image-based deep neural network assigning each peak group a probability score of being a true peak). At each score threshold cutoff value, compute the true positive rate (sensitivity) and true negative rate (specificity), then plot receiver operating characteristic curves — in this study the DNN achieved AUC 0.992 (training) and 0.988 (test). Select an optimum operating point from the ROC analysis (here, score threshold 0.4725, corresponding to a true positive rate of approximately 0.945 and a false positive rate of approximately 0.04). To compare against a conventional workflow, independently compute the fraction of manually labeled true peaks retained and the fraction of manually labeled false peaks eliminated per model — e.g., the DNN retained 88% of true peaks and eliminated 89% of false peaks under more restrictive MZmine 2 settings, where the conventional baseline captured 11,026 features with approximately 50% true and 50% false positive peaks. Write the per-model rates to a summary table such as dnn_param_filter_metrics.csv.

## Related tools

- **R** (Custom R scripts used to compute rates, ROC curves, and summary tables from peak group classifications)
- **EDML_deep_learning2.py** (Script used to train the deep neural network model whose score thresholds are evaluated for sensitivity and specificity) — JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **MZmine 2** (Generates the candidate peak groups (more/less restrictive settings) that form the labeled evaluation set)

## Examples

```
python EDML_deep_learning2.py  # train DNN, then compute ROC/TPR/FPR per threshold and write dnn_param_filter_metrics.csv in R
```

## Evaluation signals

- ROC AUC matches reported values: 0.992 on training set and 0.988 on test set
- At the optimum threshold of 0.4725, true positive rate is approximately 0.945 and false positive rate is approximately 0.04
- On the Cohort 2 more-restrictive evaluation, the DNN retains ~88% of true peaks and eliminates ~89% of false peaks; the parameter model retains ~80% and eliminates ~66%
- Conventional baseline rates for comparison: ~50% true retained and ~50% false retained across 11,026 features
- Output summary table dnn_param_filter_metrics.csv contains both per-model rates for direct comparison

## Limitations

- Sensitivity/specificity depend on the quality of manual classification of the reference peak groups; training relied on 1304 manually reviewed examples, 740 calibration and 726 test examples
- Rate metrics are specific to the peak extraction settings used (more restrictive vs less restrictive, with false positive proportion ~75% under less restrictive settings)
- The score threshold 0.4725 was selected via study-specific criteria documented in the Supporting Information and may need recalibration for other data

## Evidence
