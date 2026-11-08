---
name: lipidomics-spectrum-classification
description: Use when you have raw untargeted LC-MS lipidomics spectra (mzML or npy
  per sample) with class labels (e.g. 928 CCLE cell lines across 23 cancer types)
  and need end-to-end disease-type classification without peak picking or annotation.
when_to_use_negative:
- Input is already a processed peak table with annotated metabolites — the skill's
  rationale is end-to-end learning directly on raw spectra
- Only a single model per class is needed — the ensemble strategy exists specifically
  to remove background-category phenomena in multi-classification
- Pre-trained model reuse is required without author contact — pre-trained serum model
  access requires contacting the author
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: 'End-to-end pipeline: mzML→npy conversion, DenseNet121 ensemble training,
    prediction, and RISE feature heatmaps'
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime environment for the DeepMSProfiler package
- name: TensorFlow 2.2.0
  role: Backend for DenseNet121 training and inference
- name: Keras 2.3.1
  role: Model definition API for the DenseNet121 architecture
- name: CUDA 10.1
  role: GPU acceleration for ensemble training (CPU-only also compatible)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:1aec76a249fb18a763f6abdc5ba80bd60908be5d5d0763a108a79749a4f6aef0
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lipidomics-spectrum-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train an ensemble of DenseNet121 models on raw lipidomic LC-MS spectra (converted mzML→npy) to classify samples into disease/cancer-type categories, then extract ensemble RISE saliency heatmaps identifying disease-associated metabolite signals.

## When to use

Use when you have raw untargeted LC-MS lipidomics spectra (mzML or npy per sample) with class labels (e.g. 928 CCLE cell lines across 23 cancer types) and need end-to-end disease-type classification without peak picking or annotation.

## When NOT to use

- Input is already a processed peak table with annotated metabolites — the skill's rationale is end-to-end learning directly on raw spectra
- Only a single model per class is needed — the ensemble strategy exists specifically to remove background-category phenomena in multi-classification
- Pre-trained model reuse is required without author contact — pre-trained serum model access requires contacting the author

## Inputs

- Raw LC-MS lipidomic spectra in mzML format (auto-converted to .npy) or pre-converted .npy spectrum files
- A datalist/label file mapping each spectrum file to its class label (e.g. cancer type per CCLE cell line)

## Outputs

- Trained DenseNet121 ensemble models (10 runs)
- Predicted class labels with AUC curves and confusion matrix plots
- Per-category accuracy, sensitivity, and specificity metrics
- Ensemble RISE saliency heatmaps saved as feature_results/ensemble_RISE.npy (shape e.g. (6, 341, 341))

## How to apply

Convert each raw mzML lipidomics spectrum to npy format and build a datalist mapping each spectrum file to its class label (e.g. cell line to cancer type). Train a DenseNet121 ensemble in DeepMSProfiler with the adam optimizer at lr 1e-4, default batch size 8, repeating training 10 times (run_train) so the ensemble removes background-category phenomena in multi-classification. Run run_predict with plot_auc=True and plot_cm=True to generate AUC curves and a confusion matrix plus accuracy, sensitivity, and specificity per class. Then run run_feature followed by show_feature(mode='ensemble') to produce ensemble RISE saliency heatmaps saved to feature_results/ensemble_RISE.npy; these heatmaps highlight metabolite m/z regions for downstream network mapping (e.g. PIUMet-based metabolite-protein networks). The end-to-end design bypasses conventional peak extraction, alignment, and annotation, retaining unannotated metabolite signals.

## Related tools

- **DeepMSProfiler** (End-to-end pipeline: mzML→npy conversion, DenseNet121 ensemble training, prediction, and RISE feature heatmaps) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime environment for the DeepMSProfiler package)
- **TensorFlow 2.2.0** (Backend for DenseNet121 training and inference)
- **Keras 2.3.1** (Model definition API for the DenseNet121 architecture)
- **CUDA 10.1** (GPU acceleration for ensemble training (CPU-only also compatible))

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature -arch DenseNet121 -opt adam -lr 1e-4 -run 10 -plot_auc -plot_cm
```

## Evaluation signals

- Ensemble must reproduce reported multi-class performance — e.g. CCLE pan-cancer: 99.3% accuracy, 97.2% sensitivity, 100% specificity across 23 cancer types; lung data benchmark: AUC 0.99 and 96.1% accuracy for early-stage lung adenocarcinoma
- run_feature must save a heatmap file at feature_results/ensemble_RISE.npy with expected shape (e.g. (6, 341, 341) in the demo log)
- Training log must show 10 repeated runs: '[INFO] run model with args: DenseNet121 adam lr 1.0e-04 0' through run 9
- Heatmap-highlighted metabolites must map onto protein-metabolite networks recovering the expected shared elements (e.g. 14 shared metabolites and 3 shared proteins across the 23 CCLE cancer-type networks)
- Confusion matrix (plot_cm) should show no background-category misassignment after ensembling

## Limitations

- Pre-trained model (859 serum samples: 210 healthy, 323 lung nodules, 326 lung cancer) is for academic use only and requires contacting the author for access
- Tested environment is specific: Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1 (CPU-only compatible), CentOS 7/Windows
- Demo data are distributed via Baidu Netdisk, which may impede retrieval outside that network
- mzML inputs are auto-converted to npy; already-processed feature tables bypass the method's raw-signal rationale

## Evidence
