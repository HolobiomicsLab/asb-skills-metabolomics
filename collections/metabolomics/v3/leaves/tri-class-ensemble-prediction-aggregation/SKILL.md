---
name: tri-class-ensemble-prediction-aggregation
description: Use when raw LC-MS metabolomics data (.npy or .mzML) must be classified
  into three disease-status classes using DeepMSProfiler's pre-trained serum model,
  and ensemble-level accuracy, AUC, and confusion matrix are needed.
when_to_use_negative:
- You need to train a new model from your own data rather than run inference with
  the pre-trained serum ensemble (use run_train instead of -run_pred with -pretrain).
- Your goal is metabolite biomarker discovery or metabolite-protein network extraction
  rather than sample classification (use the run_feature/show_feature interpretability
  route, not prediction aggregation).
- The task is a binary or non-tri-class classification problem (e.g. the PACS seven-class
  image task or CCLE lipid data) where the three-class serum labels do not apply.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end framework that runs ensemble prediction on raw LC-MS data via mainRun.py
    with -run_pred and -pretrain, auto-converts .mzML to .npy, and plots the confusion
    matrix
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime environment for executing mainRun.py prediction commands
- name: TensorFlow 2.2.0
  role: Deep-learning backend running the DenseNet121 ensemble inference
- name: Keras 2.3.1
  role: Model definition/loads the DenseNet121 pre-trained ensemble weights
- name: CUDA 10.1
  role: GPU acceleration for ensemble inference (CPU-only environments also compatible)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:06ebe636eafd778b256aa4aa3e8b70fe6ba7b95a0abded46024d2c338f324d2d
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
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tri-class-ensemble-prediction-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates predictions from an ensemble of 10 DenseNet121 models trained with adam optimizer (lr 1e-04) to classify raw LC-MS serum metabolomics samples into three classes — healthy, lung nodule, and lung cancer (stage-I adenocarcinoma) — and compute tri-class accuracy/AUC and a confusion matrix.

## When to use

Use when raw LC-MS metabolomics data (.npy or .mzML) must be classified into three disease-status classes using DeepMSProfiler's pre-trained serum model, and ensemble-level accuracy, AUC, and confusion matrix are needed.

## When NOT to use

- You need to train a new model from your own data rather than run inference with the pre-trained serum ensemble (use run_train instead of -run_pred with -pretrain).
- Your goal is metabolite biomarker discovery or metabolite-protein network extraction rather than sample classification (use the run_feature/show_feature interpretability route, not prediction aggregation).
- The task is a binary or non-tri-class classification problem (e.g. the PACS seven-class image task or CCLE lipid data) where the three-class serum labels do not apply.

## Inputs

- Raw LC-MS metabolomics data files in .npy format (or .mzML, auto-converted to .npy), placed in ../example/all_data/
- Class-label text file mapping samples to tri-class labels (health / nodule / cancer), e.g. ../example/all_label.txt
- Pre-trained DenseNet121 ensemble model directory (10 runs, adam, lr 1e-04), e.g. ../example/pretrain_model

## Outputs

- Ensemble-mode tri-class sample predictions (disease type labels: health, nodule, cancer)
- Classification accuracy on the independent testing samples (reported 0.961 for stage-I lung adenocarcinoma)
- Tri-class AUC (reported at the 0.99 level) when -plot_auc is used
- Confusion matrix saved as confusion_matrix.svg in the output directory (../jobs)

## How to apply

Load the pre-trained DenseNet121 ensemble (10 runs, adam optimizer, lr 1e-04) via DeepMSProfiler's -pretrain option pointing at ../example/pretrain_model. Supply raw metabolomics data from ../example/all_data/ with class labels from ../example/all_label.txt; .mzML inputs are automatically converted to .npy before inference. Run prediction only (no training) with -run_pred in ensemble mode over the tri-class labels (health, nodule, cancer), then aggregate the 10 per-run predictions into ensemble-level class calls. Compute classification accuracy for the lung cancer (stage-I adenocarcinoma) class and plot the confusion matrix with -plot_cm, saved as confusion_matrix.svg in the output directory; optionally plot AUC with -plot_auc. The ensemble strategy is what removes background-category phenomena in the multi-class deep-learning model, so predictions must be aggregated across all 10 runs rather than taken from a single model.

## Related tools

- **DeepMSProfiler** (End-to-end framework that runs ensemble prediction on raw LC-MS data via mainRun.py with -run_pred and -pretrain, auto-converts .mzML to .npy, and plots the confusion matrix) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime environment for executing mainRun.py prediction commands)
- **TensorFlow 2.2.0** (Deep-learning backend running the DenseNet121 ensemble inference)
- **Keras 2.3.1** (Model definition/loads the DenseNet121 pre-trained ensemble weights)
- **CUDA 10.1** (GPU acceleration for ensemble inference (CPU-only environments also compatible))

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm
```

## Evaluation signals

- Accuracy on the independent testing samples reproduces the reported value of 0.961 for early-stage lung adenocarcinoma detection.
- Tri-class AUC reaches the reported 0.99 level across health / nodule / cancer classes.
- Ensemble aggregation spans all 10 DenseNet121 runs (log shows 'run model with args: DenseNet121 adam lr 1.0e-04 0' through 9) before predictions are combined.
- confusion_matrix.svg is produced in the -out directory when -plot_cm is enabled, matching the README's example result figure.
- Input samples are correctly tri-partitioned into healthy (210), lung nodule (323), and lung cancer (326) categories consistent with the pre-trained model's label space.

## Limitations

- Access to the pre-trained serum model (859 samples: 210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author; it is not bundled by default.
- Validation depends on the provided independent testing samples in ../example/all_data/; performance on new hospitals or cohorts is not guaranteed by this skill alone.
- Requires pinned versions (TensorFlow == 2.2.0, Keras == 2.3.1, Python >= 3.6, CUDA 10.1) or a CPU-only fallback; mismatched versions may break inference.

## Evidence
