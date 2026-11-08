---
name: roc-auc-metric-computation
description: Use when a trained classifier (e.g. massNet's sigmoid output layer) emits
  per-spectrum class probabilities for labeled normal/tumor MSI spectra and you must
  report ROC AUC, confusion matrix, accuracy, and runtime.
when_to_use_negative:
- Input is a small feature-engineered peak table rather than raw high-dimensional
  mass spectral data — the SVM comparison context and paper's scope are end-to-end
  classification from 85,062 m/z variables
- The classification task is multi-class rather than binary normal/tumor — the paper
  only demonstrates binary class classification and defers multi-class extension to
  future work
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning model whose sigmoid output layer supplies per-spectrum normal/tumor
    class probabilities evaluated by the ROC AUC metrics
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Framework in which the massNet architecture is implemented and run in inference
    mode to obtain predicted class probabilities
- name: tensorflow
  role: Backend for the Keras massNet model during probabilistic inference over the
    test set
- name: Python
  role: Implementation language for the metric computation (with sklearn for ROC/AUC
    and confusion-matrix calculations)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:64696d3ed83caaf06d9c8574ba39675c1f13d9c65a2fdb88ec5962d064744d04
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# roc-auc-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes ROC AUC, confusion matrix, accuracy, and per-class precision/recall/F1 over massNet's probabilistic pixel-based classifications of MALDI FT-ICR MSI spectra, to quantify tumor-vs-normal delineation performance on a withheld test set.

## When to use

Use when a trained classifier (e.g. massNet's sigmoid output layer) emits per-spectrum class probabilities for labeled normal/tumor MSI spectra and you must report ROC AUC, confusion matrix, accuracy, and runtime.

## When NOT to use

- Input is a small feature-engineered peak table rather than raw high-dimensional mass spectral data — the SVM comparison context and paper's scope are end-to-end classification from 85,062 m/z variables
- The classification task is multi-class rather than binary normal/tumor — the paper only demonstrates binary class classification and defers multi-class extension to future work

## Inputs

- Withheld MALDI FT-ICR MSI test set in HDF5 format (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables)
- Ground-truth normal/tumor labels from expert pathologist annotation of H&E images transferred to the MSI data
- Trained massNet model (VAE encoder + sigmoid classifier on 5-dimensional latent variable, hyper-parameter set H = {phi, theta}) in Keras/TensorFlow

## Outputs

- massnet_test_metrics.csv with confusion matrix, accuracy, per-class precision, recall, F1-score, and per-class ROC AUC
- roc_curve.png ROC curve plot
- Classification runtime measurement (spectra per unit time) for comparison against the SVM benchmark

## How to apply

Run the trained massNet model (Keras 2.2.0 with TensorFlow 1.8.0 backend, per the README) in inference mode over every withheld test spectrum — 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables — taking the sigmoid-activated output on the 5-dimensional latent variable as class probabilities, with no prior preprocessing or peak picking. With sklearn (0.23.2), compute the ROC curve and per-class AUC from the predicted probabilities against the pathologist-annotated ground-truth labels transferred from H&E images, plus the confusion matrix, accuracy, per-class precision, recall, and F1-score. Measure classification runtime (spectra per unit time) and benchmark against an SVM reference, where massNet reported higher accuracy and a 174-fold speed gain. Write the metrics table to massnet_test_metrics.csv and save the ROC curve as roc_curve.png; validate that metrics cover the full independent test set of 12,659 spectra.

## Related tools

- **massNet** (Deep learning model whose sigmoid output layer supplies per-spectrum normal/tumor class probabilities evaluated by the ROC AUC metrics) — https://github.com/wabdelmoula/massNet
- **Keras** (Framework in which the massNet architecture is implemented and run in inference mode to obtain predicted class probabilities)
- **tensorflow** (Backend for the Keras massNet model during probabilistic inference over the test set)
- **Python** (Implementation language for the metric computation (with sklearn for ROC/AUC and confusion-matrix calculations))

## Examples

```
import sklearn.metrics as m; auc_n = m.roc_auc_score(y_true==0, y_prob[:,0]); auc_t = m.roc_auc_score(y_true==1, y_prob[:,1]); print(m.confusion_matrix(y_true, y_pred), auc_n, auc_t)
```

## Evaluation signals

- Per-class ROC AUC values near the reported 95.01% (normal class) and 94.94% (tumor class) on the withheld test set
- Metrics computed over the full independent test set of 12,659 spectra (9,798 normal + 2,861 tumor), matching the reported accuracy of 95%
- Reproduced accuracy exceeds the SVM benchmark and classification runtime shows the reported 174-fold speed gain (~29.88 s)
- Confusion matrix dimensions are 2x2 with counts summing to 12,659, and every row corresponds to a labeled test spectrum
- Output files massnet_test_metrics.csv and roc_curve.png exist and the CSV contains accuracy, per-class precision, recall, F1-score, and AUC columns

## Limitations

- Ground truth is derived solely from annotated histology, an approximation that may not be optimal for intra-tumor heterogeneity and would require more pathological investigation at tumor margins
- Only binary (normal vs tumor) classification is demonstrated; multi-class evaluation with this metric setup is left for future development
- No software version numbers or hyperparameter values for the trained model are reported in the article text (the README supplies versions: Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0)

## Evidence
