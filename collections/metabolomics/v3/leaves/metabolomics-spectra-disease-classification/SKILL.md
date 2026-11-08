---
name: metabolomics-spectra-disease-classification
description: Use when the input is raw untargeted LC-MS metabolomic data (.npy or
  mzML per sample) with a disease-label file, and the goal is multi-class disease-state
  prediction while removing batch/hospital effects.
when_to_use_negative:
- Input is already a processed peak/feature table from conventional peak picking,
  alignment, and annotation workflows rather than raw spectra
- The task is quantifying layer-wise batch-effect removal from hidden layers (use
  the PCA + NMI/ARI/ASC/KBET/LISI layer-scan skill instead of classification)
- Only a single-model (non-ensemble) fit is acceptable, since batch/inter-hospital
  variability removal depends on the 10-run ensemble strategy
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: DeepMSProfiler
  role: End-to-end pipeline that converts raw LC-MS data to .npy, trains the ensemble
    DenseNet121, predicts disease labels, and extracts RISE feature heatmaps
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework executing the DenseNet121 ensemble training and prediction
- name: Keras 2.3.1
  role: Keras API layer over TensorFlow used to build and fit the DenseNet121 models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:220067d002c294c2810fa412c682f43f2152667b465f3af04d976dd91f362b5b
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
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# metabolomics-spectra-disease-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end deep learning classification of raw untargeted LC-MS metabolomic spectra into disease-state labels (e.g., cancer vs. nodule vs. healthy) using DeepMSProfiler's ensemble DenseNet121, bypassing conventional peak picking, alignment, and annotation.

## When to use

Use when the input is raw untargeted LC-MS metabolomic data (.npy or mzML per sample) with a disease-label file, and the goal is multi-class disease-state prediction while removing batch/hospital effects.

## When NOT to use

- Input is already a processed peak/feature table from conventional peak picking, alignment, and annotation workflows rather than raw spectra
- The task is quantifying layer-wise batch-effect removal from hidden layers (use the PCA + NMI/ARI/ASC/KBET/LISI layer-scan skill instead of classification)
- Only a single-model (non-ensemble) fit is acceptable, since batch/inter-hospital variability removal depends on the 10-run ensemble strategy

## Inputs

- Raw metabolomic spectra directory (.npy files; mzML is auto-converted to .npy)
- Label/datalist text file mapping each sample file to a disease class and a train/test split
- Optional pre-trained model directory (859-sample serum model: 210 healthy, 323 lung nodules, 326 lung cancer; requires author contact for access)

## Outputs

- Sample disease-type label predictions (run_predict)
- AUC curve and confusion matrix plots (with plot_auc/plot_cm)
- Ensemble RISE feature heatmaps saved as feature_results/ensemble_RISE.npy
- Disease-associated metabolite-protein network plots (show_feature)

## How to apply

Install DeepMSProfiler with Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1 (CUDA 10.1 or CPU-only). Provide a data directory of raw spectra plus a label file mapping each sample (e.g., C01.npy -> cancer, H01.npy -> health, N01.npy -> nodule) and a train/test split, then train an ensemble of 10 DenseNet121 runs with the adam optimizer, lr 1e-4, and batch size 8 (epochs adjustable, demo uses 2). Run prediction on the test split with AUC-curve and confusion-matrix plotting enabled, then optionally run run_feature/show_feature to obtain RISE heatmaps and metabolite-protein networks. mzML inputs are automatically converted to .npy. The ensemble strategy is what removes background-category phenomena and inter-hospital batch variability, so do not collapse to a single model run.

## Related tools

- **DeepMSProfiler** (End-to-end pipeline that converts raw LC-MS data to .npy, trains the ensemble DenseNet121, predicts disease labels, and extracts RISE feature heatmaps) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework executing the DenseNet121 ensemble training and prediction)
- **Keras 2.3.1** (Keras API layer over TensorFlow used to build and fit the DenseNet121 models)

## Examples

```
from DeepMSProfiler import *; run_train(datalist_path='DeepMSProfiler/example/datalist.txt',data_dir='DeepMSProfiler/example/data',job_dir='DeepMSProfiler/example/out/jobs007',epoch=2); run_predict(job_dir='DeepMSProfiler/example/out/jobs007',plot_auc=True,plot_cm=True)
```

## Evaluation signals

- Demo log shows 10 ensemble runs completing: '[INFO] run model with args: DenseNet121 adam lr 1.0e-04 0' through '9', followed by 'Step2 Train Done!'
- run_predict produces an AUC curve and confusion matrix; the paper reports AUC 0.99 for distinguishing lung adenocarcinoma, benign nodules, and healthy controls and accuracy 0.961 for early-stage disease
- run_feature emits '[INFO] Heatmap Shape: (6, 341, 341)' and saves feature_results/ensemble_RISE.npy
- A datalist table is printed mapping every .npy file to a Label (cancer/health/nodule) and train/test split before training begins
- Confusion-matrix and ensemble_RISE.png result files appear in the example/result outputs when running the demo

## Limitations

- The pre-trained 859-sample serum model requires contacting the author for access ('Please contact the author for access.')
- Batch-effect removal and unknown-metabolite handling are properties of the ensemble design; ablations are reported only for the model flow described in the paper
- Environment pinning is strict (TensorFlow == 2.2.0, Keras == 2.3.1, Python >= 3.6, CUDA 10.1 tested); other versions are untested, though CPU-only environments are compatible

## Evidence
