---
name: roc-auc-performance-evaluation
description: Use when evaluating a binary LC-MS peak classifier that emits true/false
  peak probability scores, by computing ROC/AUC on held-out calibration and test sets
  or by tracing performance versus training-set size.
when_to_use_negative:
- Peak groups have not yet been extracted (e.g., via MZmine 2) and converted to raster
  images with probability scores — there is nothing to evaluate with ROC/AUC yet
- The prediction task is multi-class or a continuous regression rather than binary
  true/false peak classification — the threshold-based ROC/AUC framing does not apply
- No manually reviewed ground-truth peak labels (true vs false) are available to compute
  true positive and true negative rates
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: R
  role: Custom R scripts used to compute ROC curves, AUC metrics, threshold selection,
    and the learning-curve analysis of training-set size
  repo: JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: EDML_deep_learning2.py
  role: Script for training the neural net model from scratch on the downloaded datasets
    prior to ROC/AUC evaluation
  repo: JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:7d7f80fdaedc0e021d700a5d9789cc33581a5bb2c5d998cfd5c9b45210bc9b58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# roc-auc-performance-evaluation

## Summary

Evaluates the binary LC-MS peak-classification deep neural network (true vs false spectral peak) using ROC curves, AUC, and score-threshold selection, including learning-curve analysis of model performance versus training-set size.

## When to use

Use when evaluating a binary LC-MS peak classifier that emits true/false peak probability scores, by computing ROC/AUC on held-out calibration and test sets or by tracing performance versus training-set size.

## When NOT to use

- Peak groups have not yet been extracted (e.g., via MZmine 2) and converted to raster images with probability scores — there is nothing to evaluate with ROC/AUC yet
- The prediction task is multi-class or a continuous regression rather than binary true/false peak classification — the threshold-based ROC/AUC framing does not apply
- No manually reviewed ground-truth peak labels (true vs false) are available to compute true positive and true negative rates

## Inputs

- Manually reviewed labeled peak examples (1304 training: 50% true / 50% false peaks; 740 calibration; 726 test)
- Raster-image representations of LC-MS peak groups (each pixel = intensity, each row = a single sample)
- Trained model probability scores (probability each peak group is a true peak)
- Subsampled training sets at a series of increasing sizes for the learning curve

## Outputs

- Receiver operating characteristic curves with sensitivity/specificity at each threshold
- AUC values for training, calibration, and test sets (expected ~0.992 training / 0.988 test)
- Selected optimum score threshold with associated TPR and FPR (e.g., 0.4725 → TPR ~0.945, FPR ~0.04)
- Learning-curve plot and summary table of calibration/test performance versus training-set size

## How to apply

Train the image-based deep neural network on the 1304 manually reviewed peak examples (50% true peaks, 50% false peaks) represented as raster images, then score the 740 calibration and 726 test peak examples. At each score threshold cutoff value, determine the true positive rate (sensitivity) and true negative rate (specificity), and plot receiver operating characteristic curves; the full model yields AUCs of 0.992 (training) and 0.988 (test). Select an operating score threshold — the paper selected 0.4725, corresponding to a true positive rate of ~0.945 and a false positive rate of ~0.04. For learning-curve analysis, downsample the training set at a series of increasing sizes, retrain the network on raster-image representations at each size, re-evaluate on the calibration and test sets, and plot AUC versus training-set size; high performance is achievable with fewer than 1000 peak examples, though the required size depends on the specific LC method. Implement the evaluation in R scripts alongside the deep learning code (EDML_deep_learning2.py) from the JainLab repository.

## Related tools

- **R** (Custom R scripts used to compute ROC curves, AUC metrics, threshold selection, and the learning-curve analysis of training-set size) — JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **EDML_deep_learning2.py** (Script for training the neural net model from scratch on the downloaded datasets prior to ROC/AUC evaluation) — JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py
```

## Evaluation signals

- Reproduced test-set AUC should be ~0.988 (training ~0.992); substantially lower values indicate an error in the scoring or evaluation pipeline
- The selected score threshold should yield a true positive rate of ~0.945 and a false positive rate of ~0.04 (threshold 0.4725)
- The reproduced learning curve should show high performance at training-set sizes below 1000 manually reviewed peaks, comparable to the full-training-set AUCs
- Evaluation must be computed on the held-out 740 calibration and 726 test peak examples, not on the training peaks
- Training examples should be balanced at 50% true peaks and 50% false peaks; class imbalance would bias the ROC interpretation

## Limitations

- The training-set size required for high performance depends on the specific LC method used, so the <1000-peak result may not transfer to other LC-MS methods
- Threshold selection (e.g., 0.4725) relied on the calibration set and Supporting Information criteria; the 726-example test set is the unbiased performance estimate
- AUC is only meaningful given the binary true/false peak labels; no graded peak-quality labels were used

## Evidence
