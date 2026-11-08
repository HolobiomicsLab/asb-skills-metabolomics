---
name: densenet-ensemble-training-tuning
description: Use when raw mzML/.npy LC-MS metabolomic spectra must be classified into
  disease/cancer-type labels end-to-end (no peak picking/alignment), and explainable
  disease-specific metabolite signals are needed.
when_to_use_negative:
- Input is already a peak-picked, aligned, annotated feature table rather than raw
  LC-MS spectra — DeepMSProfiler is designed for end-to-end raw-signal analysis
- A pretrained model path is supplied with -run_pred only and no training is intended;
  use prediction mode instead of retraining the ensemble
- Environment lacks TensorFlow 2.2.0 / Keras 2.3.1 compatibility and no CPU fallback
  is acceptable — version mismatches break DenseNet121 training
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end package providing run_train, run_predict, run_feature and show_feature
    for DenseNet121 ensemble training and RISE heatmap generation
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Deep learning backend for training the DenseNet121 ensemble
- name: Keras 2.3.1
  role: Model-definition layer used by DenseNet121 architecture
- name: CUDA 10.1
  role: GPU acceleration for the 10-run ensemble training (CPU-only environments also
    supported)
- name: Liquid chromatography mass spectrometry (LC-MS)
  role: Experimental technique producing the raw metabolomic spectra that serve as
    ensemble input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:fb353d891a91f73dc1bcdac21e8cd1ea60a8e1e58b20e70080a10151e72ebbdd
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
  - Liquid chromatography mass spectrometry (LC-MS)
  - PIUMet
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# densenet-ensemble-training-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train an ensemble of 10 DenseNet121 models (adam optimizer, lr 1e-4) on raw LC-MS metabolomic spectra converted to .npy format via DeepMSProfiler, then evaluate ensemble accuracy/sensitivity/specificity and generate RISE saliency heatmaps for biomarker discovery.

## When to use

Use when raw mzML/.npy LC-MS metabolomic spectra must be classified into disease/cancer-type labels end-to-end (no peak picking/alignment), and explainable disease-specific metabolite signals are needed.

## When NOT to use

- Input is already a peak-picked, aligned, annotated feature table rather than raw LC-MS spectra — DeepMSProfiler is designed for end-to-end raw-signal analysis
- A pretrained model path is supplied with -run_pred only and no training is intended; use prediction mode instead of retraining the ensemble
- Environment lacks TensorFlow 2.2.0 / Keras 2.3.1 compatibility and no CPU fallback is acceptable — version mismatches break DenseNet121 training

## Inputs

- Raw LC-MS metabolomic spectra in mzML format (auto-converted to .npy) or pre-converted .npy spectrum files
- Datalist/label file mapping each spectrum file (e.g. C01.npy, H01.npy) to its disease-type label and train/test split
- Optional pre-trained model directory (859-sample serum model: 210 healthy, 323 lung nodules, 326 lung cancer; requires author contact)

## Outputs

- Trained DenseNet121 ensemble (10 runs) under the job directory
- Per-class accuracy, sensitivity, specificity plus AUC curves and confusion matrix plots
- Ensemble RISE saliency heatmap at feature_results/ensemble_RISE.npy (e.g. shape (6, 341, 341))
- Disease-type labels for samples and heatmaps of metabolite-signal/disease correlation

## How to apply

Convert raw mzML LC-MS files to .npy and build a datalist mapping each spectrum file to its disease/cancer-type label. Train a DenseNet121 ensemble via run_train using the adam optimizer with learning rate 1e-4, repeated over 10 runs (the default -run 10; demo uses epoch=2) under TensorFlow 2.2.0 / Keras 2.3.1 / CUDA 10.1. Evaluate with run_predict using plot_auc and plot_cm to produce AUC curves and confusion matrices, and compute accuracy, sensitivity, and specificity per class. Then run run_feature and show_feature with mode='ensemble' to generate the ensemble RISE saliency heatmap saved to feature_results/ensemble_RISE.npy (demo heatmap shape 6x341x341). The rationale: the ensemble strategy removes background-category phenomena in multi-classification models and mitigates batch effects and inter-hospital variability without peak extraction steps.

## Related tools

- **DeepMSProfiler** (End-to-end package providing run_train, run_predict, run_feature and show_feature for DenseNet121 ensemble training and RISE heatmap generation) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Deep learning backend for training the DenseNet121 ensemble)
- **Keras 2.3.1** (Model-definition layer used by DenseNet121 architecture)
- **CUDA 10.1** (GPU acceleration for the 10-run ensemble training (CPU-only environments also supported))
- **Liquid chromatography mass spectrometry (LC-MS)** (Experimental technique producing the raw metabolomic spectra that serve as ensemble input)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature -arch DenseNet121 -opt adam -lr 1e-4 -run 10
```

## Evaluation signals

- Log shows exactly 10 lines of '[INFO] run model with args: DenseNet121 adam lr 1.0e-04 <run_id>' (runs 0-9) before 'Step2 Train Done!'
- run_predict outputs AUC curves and a confusion matrix; demo case reports AUC 0.99 and 96.1% accuracy for early-stage lung adenocarcinoma detection
- CCLE pan-cancer application must reproduce 23-category performance of 99.3% accuracy, 97.2% sensitivity, 100% specificity
- run_feature saves 'ensemble_RISE.npy' with demo heatmap shape (6, 341, 341); show_feature with mode='ensemble' renders class-wise heatmaps
- Datalist is parsed before training: log prints the FilePath/Label/Dataset table and 'Step1 Done!'

## Limitations

- Pre-trained model access requires contacting the author (<maintainer e-mail, see the repository README>)
- Ensemble training is compute-heavy: demo took ~25 min on two Tesla P100 GPUs for 27 files; 928 CCLE cell lines will take far longer
- Version pinning is strict (TensorFlow == 2.2.0, Keras == 2.3.1); newer stacks may fail
- Saliency heatmaps highlight metabolite signals but downstream biomarker interpretation (e.g. PIUMet networks) is a separate step

## Evidence
