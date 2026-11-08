---
name: lc-ms-serum-metabolomics-classification
description: Use when the task is to classify raw untargeted LC-MS serum samples (mzML
  or .npy) into tri-class disease labels — healthy / lung nodule / lung cancer — or
  to reproduce the 0.961 early-stage lung adenocarcinoma accuracy via a pre-trained
  ensemble model.
when_to_use_negative:
- The task requires annotated metabolite IDs or conventional peak picking, alignment
  and annotation workflows — DeepMSProfiler is designed to process raw untargeted
  LC-MS signals end-to-end instead
- Input is already a processed feature/peak table rather than raw LC-MS signals, since
  the method converts raw data directly into class outputs without peak extraction
- The target disease cohort is outside the serum metabolomics tri-class setting the
  pre-trained model covers (healthy / lung nodule / lung cancer); training a new model
  via -run_train would be needed rather than -run_pred with -pretrain
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end framework that ingests raw LC-MS data, runs the DenseNet121 ensemble
    (pretrain + run_pred), and emits class labels, confusion matrix and AUC plots
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime environment for DeepMSProfiler scripts (mainRun.py, showFeature.py)
- name: TensorFlow 2.2.0
  role: Deep learning backend executing the DenseNet121 ensemble model inference
- name: Keras 2.3.1
  role: Model definition/training API used with TensorFlow 2.2.0 for the DenseNet121
    ensemble
- name: CUDA 10.1
  role: GPU acceleration for ensemble inference; CPU-only environments are also compatible
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:c50f3726a4a94cde14da6005b89d52a354cbc4e5438e9659cfeea34593e03467
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lc-ms-serum-metabolomics-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies disease states (healthy, lung nodule, stage-I lung adenocarcinoma) directly from raw LC-MS serum metabolomics data using the DeepMSProfiler tri-class DenseNet121 ensemble, bypassing conventional peak picking, alignment and annotation. Reported performance is 96.1% accuracy and AUC 0.99 on independent testing samples.

## When to use

Use when the task is to classify raw untargeted LC-MS serum samples (mzML or .npy) into tri-class disease labels — healthy / lung nodule / lung cancer — or to reproduce the 0.961 early-stage lung adenocarcinoma accuracy via a pre-trained ensemble model.

## When NOT to use

- The task requires annotated metabolite IDs or conventional peak picking, alignment and annotation workflows — DeepMSProfiler is designed to process raw untargeted LC-MS signals end-to-end instead
- Input is already a processed feature/peak table rather than raw LC-MS signals, since the method converts raw data directly into class outputs without peak extraction
- The target disease cohort is outside the serum metabolomics tri-class setting the pre-trained model covers (healthy / lung nodule / lung cancer); training a new model via -run_train would be needed rather than -run_pred with -pretrain

## Inputs

- Raw LC-MS serum metabolomics data (.mzML files, auto-converted to .npy, or .npy files directly) in a data directory (e.g. ../example/all_data/)
- Sample label file mapping files to tri-class disease labels (e.g. ../example/all_label.txt; labels: health, nodule, cancer)
- Pre-trained DenseNet121 ensemble model directory (e.g. ../example/pretrain_model)

## Outputs

- Tri-class sample disease type labels (ensemble predictions over healthy / lung nodule / lung cancer)
- Classification accuracy on independent testing samples (target: 0.961 for stage-I lung adenocarcinoma)
- Confusion matrix plot saved as confusion_matrix.svg in the output directory (with -plot_cm)
- AUC curve (with -plot_auc; reported tri-class AUC 0.99)

## How to apply

Load the pre-trained DenseNet121 ensemble (10 runs, adam optimizer, lr 1e-04) with DeepMSProfiler's -pretrain option pointing to ../example/pretrain_model. Provide raw metabolomics data (mzML files are auto-converted to .npy) plus a label file listing tri-class sample labels, then run the prediction step: python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm, which runs ensemble-mode inference over the tri-class labels. Aggregate the ensemble predictions on the testing samples and compute classification accuracy for the lung cancer (stage-I adenocarcinoma) class; plot the confusion matrix with -plot_cm and optionally the AUC curve with -plot_auc. Correct application is judged by whether the accuracy on independent testing samples reproduces the reported 0.961 for early-stage lung adenocarcinoma, with tri-class AUC at the reported 0.99 level. The underlying model was pre-trained on 859 serum metabolomics samples (210 healthy, 323 lung nodules, 326 lung cancer).

## Related tools

- **DeepMSProfiler** (End-to-end framework that ingests raw LC-MS data, runs the DenseNet121 ensemble (pretrain + run_pred), and emits class labels, confusion matrix and AUC plots) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime environment for DeepMSProfiler scripts (mainRun.py, showFeature.py))
- **TensorFlow 2.2.0** (Deep learning backend executing the DenseNet121 ensemble model inference)
- **Keras 2.3.1** (Model definition/training API used with TensorFlow 2.2.0 for the DenseNet121 ensemble)
- **CUDA 10.1** (GPU acceleration for ensemble inference; CPU-only environments are also compatible)

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm
```

## Evaluation signals

- Accuracy on the independent testing samples should reproduce the reported 0.961 for detecting stage-I lung adenocarcinoma
- Tri-class AUC should be at the reported 0.99 level (verify with -plot_auc)
- Confusion matrix (confusion_matrix.svg) should show the lung cancer (stage-I adenocarcinoma) class dominating correct predictions with few misclassifications against benign lung nodules and healthy controls
- All samples in the label file should receive a tri-class ensemble prediction (no missing labels) across the 10-run ensemble
- Environment must satisfy TensorFlow == 2.2.0, Keras == 2.3.1, Python >= 3.6; predictions from -pretrain mode should not require retraining (no -run_train)

## Limitations

- The pre-trained model (859 serum metabolomics samples: 210 healthy, 323 lung nodules, 326 lung cancer) is for academic use only — 'Please contact the author for access' (<maintainer e-mail, see the repository README>)
- Demo data is distributed via Baidu Netdisk (password acaa); download availability outside that mirror is not documented
- mzML inputs are auto-converted to .npy, but the documented demo files are already in .npy format, so conversion behavior on very large raw files is unverified here
- Tested on CentOS 7 with CUDA 10.1 (TensorFlow 2.2.0 / Keras 2.3.1 are pinned older versions); CPU-only environments are compatible but runtime behavior is not characterized

## Evidence
