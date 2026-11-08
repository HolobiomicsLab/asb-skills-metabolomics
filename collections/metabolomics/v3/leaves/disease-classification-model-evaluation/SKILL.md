---
name: disease-classification-model-evaluation
description: Use when a re-trained or pre-trained DeepMSProfiler model must be scored
  on an independent LC-MS testing set (e.g., MetaboLights MTBLS1129) and its metrics
  compared against published values.
when_to_use_negative:
- Input is already a peak-picked, aligned, and annotated feature table rather than
  raw mzML/.npy LC-MS signals — the end-to-end model expects raw spectra
- The goal is biomarker/interpretability analysis (metabolite-protein networks or
  RISE heatmaps) rather than scoring held-out classification performance, which is
  handled by run_feature/show_feature
- There is no labeled independent testing split — metrics on the training data would
  not validate generalization
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Provides run_predict (with -plot_auc/-plot_cm) to produce ensemble test-set
    predictions, metrics, AUC curve, and confusion matrix; installed via pip or git
    clone
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework on which the DenseNet121 ensemble models run during training
    and test prediction
- name: Keras 2.3.1
  role: Keras API layer used with TensorFlow 2.2.0 to build and evaluate the classifier
- name: CUDA 10.1
  role: GPU capability for training/prediction, though the pipeline is also compatible
    with CPU-only environments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:3898d37567d8e68171168fce5076ed3e4686d384a2c0ea44b8610062e552601a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - CUDA
  - CUDA 10.1
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# disease-classification-model-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates an ensemble deep-learning disease classifier (DeepMSProfiler) on a held-out independent testing split of raw LC-MS metabolomics data, reporting accuracy, precision, recall, F1, AUC, and a confusion matrix to confirm reported performance.

## When to use

Use when a re-trained or pre-trained DeepMSProfiler model must be scored on an independent LC-MS testing set (e.g., MetaboLights MTBLS1129) and its metrics compared against published values.

## When NOT to use

- Input is already a peak-picked, aligned, and annotated feature table rather than raw mzML/.npy LC-MS signals — the end-to-end model expects raw spectra
- The goal is biomarker/interpretability analysis (metabolite-protein networks or RISE heatmaps) rather than scoring held-out classification performance, which is handled by run_feature/show_feature
- There is no labeled independent testing split — metrics on the training data would not validate generalization

## Inputs

- Raw LC-MS metabolomics files in mzML format (auto-converted to .npy by DeepMSProfiler)
- Sample label file assigning disease-type labels and the 4:1 discovery/testing split (e.g., datalist.txt / label.txt)
- Trained ensemble model directory (e.g., jobs/jobs007 from run_train, or a pre-trained model obtained from the author)

## Outputs

- test_metrics.csv with accuracy, precision, recall, and F1 on the independent test set
- confusion_matrix.svg from -plot_cm
- AUC curve plot from -plot_auc
- Ensemble per-sample disease-type predictions on the held-out testing split

## How to apply

Split the labeled samples (e.g., the 236-sample MTBLS1129 colon cancer LC-MS dataset) into discovery and testing sets at the documented 4:1 ratio, recording assignments in the sample label file. Train the 10-run DenseNet121 ensemble on the discovery split with the adam optimizer, learning rate 1e-4, and batch size 8, then run ensemble prediction on the held-out test split with run_predict, enabling -plot_auc and -plot_cm so the AUC curve and confusion matrix (confusion_matrix.svg) are generated. Compute accuracy, precision, recall, and F1 from the ensemble predictions on the test set and write them to test_metrics.csv. Validate by comparing against reported values — accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8% for MTBLS1129 — allowing only small run-to-run variation.

## Related tools

- **DeepMSProfiler** (Provides run_predict (with -plot_auc/-plot_cm) to produce ensemble test-set predictions, metrics, AUC curve, and confusion matrix; installed via pip or git clone) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework on which the DenseNet121 ensemble models run during training and test prediction)
- **Keras 2.3.1** (Keras API layer used with TensorFlow 2.2.0 to build and evaluate the classifier)
- **CUDA 10.1** (GPU capability for training/prediction, though the pipeline is also compatible with CPU-only environments)

## Examples

```
from DeepMSProfiler import *; run_predict(job_dir='DeepMSProfiler/example/out/jobs007', plot_auc=True, plot_cm=True)
```

## Evaluation signals

- test_metrics.csv on the MTBLS1129 independent test set reproduces accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8% (small run-to-run variation permitted)
- confusion_matrix.svg is produced after run_predict with -plot_cm enabled
- AUC curve is generated when -plot_auc is enabled; the lung-cancer model reports AUC 0.99 and accuracy 0.961 as reference points
- Predictions come from the ensemble of 10 independently trained DenseNet121 runs (DenseNet121, adam, lr 1.0e-04, batch size 8), as seen in the demo log run lines 0–9
- The testing split is strictly held out from the 4:1 discovery/testing split and never used in training

## Limitations

- The pre-trained model (859 serum metabolomics samples: 210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author for access
- Run-to-run variation exists across the 10-run ensemble, so exact metric equality with published values should not be expected
- Dependency pins are strict (Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1); CPU-only environments are supported but slower

## Evidence
