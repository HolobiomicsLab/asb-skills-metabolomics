---
name: early-stage-lung-adenocarcinoma-detection
description: Use when raw LC-MS serum metabolomics data (mzML or .npy) must be classified
  into lung cancer / benign lung nodule / healthy groups, and the target is early-stage
  (stage-I) adenocarcinoma detection accuracy.
when_to_use_negative:
- The input is already a peak-picked, aligned, annotated feature table — DeepMSProfiler
  is designed for end-to-end analysis of raw metabolic signals without peak extraction/alignment
- The goal is biomarker/feature interpretation (metabolite-protein networks) rather
  than sample classification — use run_feature/show_feature RISE heatmaps instead
  of prediction-only mode
- Only the public demo data is available and the pre-trained serum model has not been
  obtained from the author — training from scratch on the small demo set will not
  reproduce the 0.961 figure
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end deep-learning pipeline (DenseNet121 ensemble) that consumes raw
    LC-MS serum data and outputs tri-class disease predictions and a confusion matrix
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Backend (pinned == 2.2.0) running the DenseNet121 ensemble inference
- name: Keras
  role: Model API layer (pinned == 2.3.1) over TensorFlow for the pre-trained ensemble
- name: CUDA
  role: GPU acceleration (CUDA 10.1) for ensemble inference, though CPU-only environments
    are compatible
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:1043af7b6bc93933205205b5f74bcdc8a4719c5acb11d28adb798de3b74b36f4
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
  - DeepMSProfiler
  - Keras
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# early-stage-lung-adenocarcinoma-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce DeepMSProfiler's tri-class ensemble (DenseNet121, 10 runs, adam, lr 1e-04) inference on raw LC-MS serum data to detect stage-I lung adenocarcinoma, which the paper reports at 96.1% accuracy with AUC 0.99 on independent testing samples.

## When to use

Use when raw LC-MS serum metabolomics data (mzML or .npy) must be classified into lung cancer / benign lung nodule / healthy groups, and the target is early-stage (stage-I) adenocarcinoma detection accuracy.

## When NOT to use

- The input is already a peak-picked, aligned, annotated feature table — DeepMSProfiler is designed for end-to-end analysis of raw metabolic signals without peak extraction/alignment
- The goal is biomarker/feature interpretation (metabolite-protein networks) rather than sample classification — use run_feature/show_feature RISE heatmaps instead of prediction-only mode
- Only the public demo data is available and the pre-trained serum model has not been obtained from the author — training from scratch on the small demo set will not reproduce the 0.961 figure

## Inputs

- Raw serum LC-MS metabolomics data in mzML format (auto-converted to .npy) or pre-converted .npy files
- Label file mapping samples to tri-class labels (healthy / lung nodule / lung cancer), e.g. ../example/all_label.txt
- Pre-trained DenseNet121 ensemble model (10 runs, adam optimizer, lr 1e-04) from ../example/pretrain_model

## Outputs

- Ensemble class predictions for each independent testing sample (cancer / nodule / health)
- Classification accuracy on the testing set (expected ~0.961 for stage-I lung adenocarcinoma)
- Confusion matrix plot saved as confusion_matrix.svg in the output directory

## How to apply

Obtain the pre-trained serum model (859 samples: 210 healthy, 323 lung nodules, 326 lung cancer) via the -pretrain option pointing to ../example/pretrain_model; note the README requires contacting the author for access. Load the independent testing metabolomics data from ../example/all_data/ with class labels from ../example/all_label.txt, letting the script auto-convert mzML files to .npy if raw spectra are supplied. Run prediction in ensemble mode with confusion-matrix plotting: python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm. Aggregate the 10-run DenseNet121 ensemble predictions over the tri-class labels and compute classification accuracy on the independent testing samples; the ensemble strategy exists to remove background-category phenomena in multi-classification deep-learning models. Render the confusion matrix (saved as confusion_matrix.svg) as the primary validation artifact against the reported 0.961 accuracy.

## Related tools

- **DeepMSProfiler** (End-to-end deep-learning pipeline (DenseNet121 ensemble) that consumes raw LC-MS serum data and outputs tri-class disease predictions and a confusion matrix) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Backend (pinned == 2.2.0) running the DenseNet121 ensemble inference)
- **Keras** (Model API layer (pinned == 2.3.1) over TensorFlow for the pre-trained ensemble)
- **CUDA** (GPU acceleration (CUDA 10.1) for ensemble inference, though CPU-only environments are compatible)

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm
```

## Evaluation signals

- Accuracy on the independent testing samples reproduces the reported 0.961 for stage-I lung adenocarcinoma detection
- Tri-class AUC is at the reported 0.99 level
- Confusion matrix (confusion_matrix.svg) is generated via -plot_cm and shows the expected cancer/nodule/health distribution
- Prediction runs in ensemble mode over 10 DenseNet121 runs rather than a single model, as in the demo log pattern 'run model with args: DenseNet121 adam lr 1.0e-04 0..9'
- Environment matches pinned versions: Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1

## Limitations

- The pre-trained serum model (859 samples) requires contacting the author for access — it is not bundled in the repo
- Demo data must be downloaded from Baidu Netdisk (password acaa) if the example .npy files are unavailable
- Training the model from scratch (e.g. on the small demo set with default epoch=2) will not reproduce the reported 0.961 early-stage accuracy; the figure is specific to the pre-trained serum ensemble on the independent testing samples

## Evidence
