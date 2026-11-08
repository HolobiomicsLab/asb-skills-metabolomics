---
name: serum-lcms-spectral-preprocessing
description: Use when input is raw untargeted LC-MS metabolomic spectra (mzML or .npy)
  from serum samples and the goal is disease-state classification plus interpretable
  disease-associated metabolite signals, without peak picking or annotation.
when_to_use_negative:
- Input is already a processed peak table or annotated metabolite feature matrix —
  DeepMSProfiler is designed to consume raw metabolic signals end-to-end
- Data are not raw LC-MS metabolomic spectra (e.g. other omics modalities or image
  data unrelated to the metabolomic profiling task)
- A single-model classifier is required rather than an ensemble, since the paper's
  reported accuracy depends on the 10-run DenseNet121 ensemble
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3473
tools:
- name: DeepMSProfiler
  role: Runs mzML-to-npy conversion, DenseNet121 ensemble training, ensemble prediction,
    and RISE feature-map extraction on raw LC-MS data
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework on which the DenseNet121 ensemble models are trained and
    evaluated
- name: Keras 2.3.1
  role: Model-definition layer used with TensorFlow for the DenseNet121 architectures
- name: CUDA 10.1
  role: GPU acceleration for ensemble training; CPU-only environments are also supported
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:007bd0c17308efd0cd8c5ecf8a310acb0d11220f62a12e6eec7a514c6f260d6c
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# serum-lcms-spectral-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end preprocessing and deep-learning classification of raw serum LC-MS spectra: convert .mzML to .npy, train an ensemble of 10 DenseNet121 models, and predict tri-class disease labels (healthy / benign nodule / lung adenocarcinoma) with ~0.99 AUC.

## When to use

Use when input is raw untargeted LC-MS metabolomic spectra (mzML or .npy) from serum samples and the goal is disease-state classification plus interpretable disease-associated metabolite signals, without peak picking or annotation.

## When NOT to use

- Input is already a processed peak table or annotated metabolite feature matrix — DeepMSProfiler is designed to consume raw metabolic signals end-to-end
- Data are not raw LC-MS metabolomic spectra (e.g. other omics modalities or image data unrelated to the metabolomic profiling task)
- A single-model classifier is required rather than an ensemble, since the paper's reported accuracy depends on the 10-run DenseNet121 ensemble

## Inputs

- Raw serum LC-MS spectra as .npy arrays (or .mzML files, auto-converted to .npy by the script)
- Label file mapping each sample to disease type (healthy / nodule / cancer) with train/test split, e.g. example/label.txt or datalist.txt

## Outputs

- Sample disease-type label predictions for the held-out test split
- AUC curve and confusion matrix figures for three-class classification
- Ensemble RISE saliency heatmap saved as feature_results/ensemble_RISE.npy
- Disease-associated metabolite-protein network plots derived from the saliency signals

## How to apply

Point the DeepMSProfiler entry point at a directory of raw spectra and a label file mapping each sample to healthy/nodule/cancer; mzML files are automatically converted to .npy. Train an ensemble of 10 DenseNet121 models with the Adam optimizer at learning rate 1e-4 and batch size 8, then run prediction in ensemble mode (-run_pred -mode ensemble), enabling -plot_auc and -plot_cm for three-class evaluation. Optionally run feature extraction (-run_feature) to save the ensemble RISE saliency heatmap (ensemble_RISE.npy) under the job's feature_results directory. The ensemble strategy is what removes background-category phenomena and inter-hospital batch effects, so single-model predictions should not be substituted.

## Related tools

- **DeepMSProfiler** (Runs mzML-to-npy conversion, DenseNet121 ensemble training, ensemble prediction, and RISE feature-map extraction on raw LC-MS data) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework on which the DenseNet121 ensemble models are trained and evaluated)
- **Keras 2.3.1** (Model-definition layer used with TensorFlow for the DenseNet121 architectures)
- **CUDA 10.1** (GPU acceleration for ensemble training; CPU-only environments are also supported)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature
```

## Evaluation signals

- Test-set AUC approximates 0.99 and accuracy ~0.95 (precision 0.96, recall 0.95, F1 0.98) on the 859-sample cross-hospital serum dataset
- Per-class accuracies approximate 85.7% (LUAD), 90.8% (benign nodule), and 97.0% (healthy), matching the published confusion matrix
- Training log shows 10 sequential model runs ('run model with args: DenseNet121 adam lr 1.0e-04 0..9')
- Feature step writes a heatmap of shape like (6, 341, 341) to feature_results/ensemble_RISE.npy
- Prediction step produces both AUC-curve and confusion-matrix figure files when -plot_auc and -plot_cm are enabled

## Limitations

- The pre-trained serum model (859 samples: 210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author for access
- Developed/tested with pinned versions (Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, CUDA 10.1) on CentOS 7; other environments are untested
- Performance depends on the ensemble; ablation evidence for overcoming inter-hospital variability is tied to the ensemble strategy

## Evidence
