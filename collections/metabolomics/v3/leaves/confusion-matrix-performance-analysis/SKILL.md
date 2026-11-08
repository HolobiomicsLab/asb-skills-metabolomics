---
name: confusion-matrix-performance-analysis
description: Use when a probabilistic pixel-based classifier has been applied to a
  labeled withheld test set (e.g., 9,798 normal and 2,861 tumor MALDI FT-ICR MSI spectra)
  and you must quantify accuracy, ROC AUC, and runtime against a benchmark such as
  SVM.
when_to_use_negative:
- The goal is training-stability assessment rather than held-out test evaluation —
  the paper used 5-fold cross validation on the training set for that purpose, which
  is a different procedure.
- The spectra require preprocessing or peak picking before classification — massNet
  classification is explicitly performed without prior preprocessing or peak picking.
- The task is multi-class tissue classification — the demonstrated model covers only
  binary normal vs tumor classification.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: sklearn
  role: Computes the confusion matrix, accuracy, per-class precision/recall/F1-score,
    and per-class AUC from the ROC curve on predicted test-set probabilities
- name: Keras
  role: Hosts the trained massNet model (VAE encoder plus sigmoid classifier) through
    which test spectra are inferred to obtain class probabilities
  repo: https://github.com/wabdelmoula/massNet
- name: tensorflow
  role: Backend executing the probabilistic inference over the 12,659 withheld test
    spectra
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Scripting environment (3.6.12 per README) in which the evaluation pipeline
    and metrics export are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:fc73639e6d8cf39c3c8b9920b460812b2cc110507fce9a582a4af02bb19be673
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
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# confusion-matrix-performance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained binary classifier (e.g., massNet's sigmoid-activated classifier on the 5-dimensional VAE latent variable) on a withheld MALDI FT-ICR MSI test set by computing a confusion matrix, accuracy, per-class precision/recall/F1, per-class ROC AUC, and classification runtime.

## When to use

Use when a probabilistic pixel-based classifier has been applied to a labeled withheld test set (e.g., 9,798 normal and 2,861 tumor MALDI FT-ICR MSI spectra) and you must quantify accuracy, ROC AUC, and runtime against a benchmark such as SVM.

## When NOT to use

- The goal is training-stability assessment rather than held-out test evaluation — the paper used 5-fold cross validation on the training set for that purpose, which is a different procedure.
- The spectra require preprocessing or peak picking before classification — massNet classification is explicitly performed without prior preprocessing or peak picking.
- The task is multi-class tissue classification — the demonstrated model covers only binary normal vs tumor classification.

## Inputs

- Withheld test set of MALDI FT-ICR MSI spectra in HDF5 format (12,659 spectra; 9,798 normal, 2,861 tumor; 85,062 m/z variables)
- Ground-truth normal/tumor labels transferred from expert pathologist annotations of consecutive H&E-stained sections
- Trained massNet model (VAE encoder plus sigmoid-activated classifier on the 5-dimensional latent variable) implemented in Keras/TensorFlow
- Predicted class probabilities for each test spectrum from the sigmoid output layer

## Outputs

- Confusion matrix over the full independent test set
- Overall accuracy and per-class precision, recall, and F1-score
- Per-class ROC AUC values and ROC curve image (roc_curve.png)
- Classification runtime (spectra per unit time) with SVM speed comparison
- Metrics table written to massnet_test_metrics.csv

## How to apply

Load the withheld test set in HDF5 format — for massNet this is 12,659 MALDI FT-ICR MSI spectra with 85,062 m/z variables (9,798 normal, 2,861 tumor) plus ground-truth labels transferred from pathologist-annotated H&E images. Run probabilistic inference on every test spectrum through the sigmoid output layer of the trained model without any prior preprocessing or peak picking, obtaining predicted class probabilities for normal vs tumor. Compute the confusion matrix, overall accuracy, per-class precision, recall, F1-score, and per-class AUC from the ROC curve over the full independent test set. Measure classification runtime (spectra per unit time) and compare against the support vector machine benchmark, where a 174-fold speed gain and higher accuracy were reported. Write the metrics table to massnet_test_metrics.csv and save the ROC curve as roc_curve.png.

## Related tools

- **sklearn** (Computes the confusion matrix, accuracy, per-class precision/recall/F1-score, and per-class AUC from the ROC curve on predicted test-set probabilities)
- **Keras** (Hosts the trained massNet model (VAE encoder plus sigmoid classifier) through which test spectra are inferred to obtain class probabilities) — https://github.com/wabdelmoula/massNet
- **tensorflow** (Backend executing the probabilistic inference over the 12,659 withheld test spectra) — https://github.com/wabdelmoula/massNet
- **Python** (Scripting environment (3.6.12 per README) in which the evaluation pipeline and metrics export are implemented)

## Examples

```
from sklearn.metrics import confusion_matrix, accuracy_score, roc_auc_score; cm = confusion_matrix(y_true, y_prob > 0.5); auc_norm = roc_auc_score(y_true, y_prob)
```

## Evaluation signals

- Verify metrics are computed over the full independent test set of 12,659 spectra (9,798 normal + 2,861 tumor), not a subsample
- Reproduced ROC AUC should be approximately 95.01% for the normal class and 94.94% for the tumor class
- Reproduced accuracy should exceed the support vector machine benchmark reported in cross-validation
- Classification runtime should be consistent with the reported 174-fold gain in speed over SVM (task card cites 29.88 s)
- Confusion matrix classes must match the ground-truth pathologist-annotated labels, with two classes (normal, tumor) only

## Limitations

- Only binary (normal vs tumor) classification is demonstrated; the authors note the model could be extended to multi-class classification in future development.
- Ground-truth annotation based solely on annotated histology is an approximation that may not be optimal, especially for intra-tumor heterogeneity.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text, creating a reproducibility gap for exact metric reproduction.

## Evidence
