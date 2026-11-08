---
name: train-calibration-test-split-evaluation
description: Use when a labeled peak dataset (e.g., 1304 training / 740 calibration
  / 726 test manually reviewed Cohort 1 peaks) must be split, a CNN classifier trained,
  and AUC/ROC performance with an operating threshold reported.
when_to_use_negative:
- Input is an unlabeled peak list with no manual true/false classifications, since
  supervised training, calibration, and AUC evaluation all require reviewed labels
- The goal is baseline peak filtering with logistic regression or Random Forest on
  59 peak group parameters rather than image-based CNN classification
- Only a single pooled dataset is available and a held-out test set cannot be formed
  without leaking training/calibration examples
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: EDML_deep_learning2.py
  role: Author-provided script used to train the convolutional neural network on the
    rasterized peak images
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: R
  role: Custom R scripts generate stacked peak plots and raster images from MZmine
    2 peak data
- name: MZmine 2
  role: Produces non-filtered peak lists from raw mzXML data prior to image construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:dc375ef796f00c7ef85098239f98da57197e7314c8a4a59605ed5221a52cae41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# train-calibration-test-split-evaluation

## Summary

Splits manually reviewed LC-MS peak examples into training, calibration, and held-out test sets, then trains the image-based deep neural network and evaluates it via ROC/AUC analysis with an optimum score threshold selection.

## When to use

Use when a labeled peak dataset (e.g., 1304 training / 740 calibration / 726 test manually reviewed Cohort 1 peaks) must be split, a CNN classifier trained, and AUC/ROC performance with an operating threshold reported.

## When NOT to use

- Input is an unlabeled peak list with no manual true/false classifications, since supervised training, calibration, and AUC evaluation all require reviewed labels
- The goal is baseline peak filtering with logistic regression or Random Forest on 59 peak group parameters rather than image-based CNN classification
- Only a single pooled dataset is available and a held-out test set cannot be formed without leaking training/calibration examples

## Inputs

- Manually reviewed Cohort 1 peak examples with true/false labels (1304 training, 740 calibration, 726 test)
- mzXML-derived peak data downloaded from https://doi.org/doi:10.25345/C5FD2F
- 64x64 raster peak-shape images generated from stacked peak plots of the 63 most-intense samples per feature

## Outputs

- Trained and calibrated deep neural network model assigning true-peak probability scores
- ROC curves for training and test sets
- dnn_peak_classification_metrics.csv containing training AUC (0.992), test AUC (0.988), and optimum threshold metrics (threshold 0.4725, TPR ~0.945, FPR ~0.04)

## How to apply

Partition the manually reviewed peak examples into disjoint training (1304 peaks, 50% true / 50% false), calibration (740 peaks), and held-out test (726 peaks) sets. Convert each peak group into a 64x64 raster image built from stacked peak plots of the 63 samples where the candidate peak was most intense, with each pixel encoding intensity, each row a single sample, and each column the time/scans dimension. Train the convolutional neural network using EDML_deep_learning2.py from the JainLab repository on the training images, then calibrate on the calibration set. Score each held-out peak group with a probability of being a true peak, compute ROC curves by determining true positive rate and false positive rate at every score threshold cutoff, and report training and test AUC (0.992 and 0.988 expected). Select the optimum operating point (score threshold 0.4725, TPR ~0.945, FPR ~0.04) and write metrics to dnn_peak_classification_metrics.csv with a rendered ROC curve.

## Related tools

- **EDML_deep_learning2.py** (Author-provided script used to train the convolutional neural network on the rasterized peak images) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **R** (Custom R scripts generate stacked peak plots and raster images from MZmine 2 peak data)
- **MZmine 2** (Produces non-filtered peak lists from raw mzXML data prior to image construction)

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Reproduced training AUC is approximately 0.992 and held-out test AUC approximately 0.988
- Selected optimum score threshold is 0.4725, yielding TPR ~0.945 and FPR ~0.04
- Training set composition matches 1304 peaks with 50% true and 50% false; calibration 740; test 726, all disjoint
- Each input image is a 64x64 raster built from exactly 63 samples (the samples where the candidate peak was most intense)
- ROC curves computed at every score threshold cutoff show true positive rate and false positive rate per threshold

## Limitations

- Model performance is tied to manually reviewed Cohort 1 peaks; reproducibility on other cohorts (e.g., Cohort 2, N = 526) requires applying the pre-trained model rather than re-splitting
- No specific framework or version for the neural network implementation is named; the script EDML_deep_learning2.py is provided without version details
- Threshold of 0.4725 was selected per Supporting Information selection criteria, so reproducing the exact operating point may require consulting the SI

## Evidence
