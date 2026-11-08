---
name: rise-saliency-heatmap-generation
description: Use when you have a trained DeepMSProfiler ensemble (e.g., 10 DenseNet121
  runs on raw .npy LC-MS spectra) and need to visualize which metabolite signal regions
  contribute to each class (cancer/nodule/healthy) before network mapping.
when_to_use_negative:
- Input is already a peak-picked, aligned feature table — DeepMSProfiler is designed
  for end-to-end analysis of raw metabolomic signals
- You only need class predictions and do not intend to interpret or map disease-associated
  metabolite regions (skip run_feature and use -run_pred with -plot_auc/-plot_cm)
- No trained ensemble or pre-trained model is available and you cannot contact the
  author for the 859-sample serum pre-trained model
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepMSProfiler
  role: Provides run_feature and show_feature (mode='ensemble') entry points that
    compute and render the RISE saliency heatmap from the trained DenseNet121 ensemble
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Backend (pinned 2.2.0) on which the ensemble models run during saliency extraction
- name: Keras
  role: Model API (pinned 2.3.1) used to load the DenseNet121 ensemble for feature
    extraction
- name: PIUMet
  role: 'Downstream: maps heatmap-highlighted metabolites onto protein-metabolite
    interaction networks to recover disease-specific metabolite-protein networks'
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:c0ffcb8b34d4debb952157c9ebfcfca823a5dad1b0a0db575a39f029b936824c
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
  - Liquid chromatography mass spectrometry (LC-MS)
  - PIUMet
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# rise-saliency-heatmap-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates RISE saliency heatmaps from a trained DeepMSProfiler DenseNet121 ensemble to localize which raw LC-MS spectral regions drive each disease-class prediction. It is the interpretability bridge that converts ensemble model attention into metabolite-protein network candidates.

## When to use

Use when you have a trained DeepMSProfiler ensemble (e.g., 10 DenseNet121 runs on raw .npy LC-MS spectra) and need to visualize which metabolite signal regions contribute to each class (cancer/nodule/healthy) before network mapping.

## When NOT to use

- Input is already a peak-picked, aligned feature table — DeepMSProfiler is designed for end-to-end analysis of raw metabolomic signals
- You only need class predictions and do not intend to interpret or map disease-associated metabolite regions (skip run_feature and use -run_pred with -plot_auc/-plot_cm)
- No trained ensemble or pre-trained model is available and you cannot contact the author for the 859-sample serum pre-trained model

## Inputs

- Trained DeepMSProfiler ensemble job directory (10 DenseNet121 runs, e.g., out/jobs007)
- Raw LC-MS spectra (.npy arrays; .mzML files are auto-converted to .npy)
- Sample disease-type label file mapping each spectrum to its class (e.g., cancer/health/nodule)

## Outputs

- ensemble_RISE.npy saliency heatmap saved under <job_dir>/feature_results/
- Rendered per-class heatmap images via show_feature (mode='ensemble')

## How to apply

After run_train and run_predict have populated a job directory (e.g., DeepMSProfiler/example/out/jobs007), execute run_feature on that job_dir to extract saliency from the 10-model DenseNet121 ensemble (adam, lr 1e-4). The output heatmap is saved as feature_results/ensemble_RISE.npy; for the demo data the expected shape is (6, 341, 341), reflecting spectral regions rather than individual m/z peaks. Then call show_feature with mode='ensemble' to render per-class heatmaps so salient signals can be inspected and, in the CCLE/pan-cancer workflow, mapped onto PIUMet protein-metabolite interaction networks. The rationale is to retain unannotated/unknown metabolite signals: the interpretability step 'enables direct access to disease-related metabolite-protein networks' without prior peak picking or identification.

## Related tools

- **DeepMSProfiler** (Provides run_feature and show_feature (mode='ensemble') entry points that compute and render the RISE saliency heatmap from the trained DenseNet121 ensemble) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Backend (pinned 2.2.0) on which the ensemble models run during saliency extraction)
- **Keras** (Model API (pinned 2.3.1) used to load the DenseNet121 ensemble for feature extraction)
- **PIUMet** (Downstream: maps heatmap-highlighted metabolites onto protein-metabolite interaction networks to recover disease-specific metabolite-protein networks)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature
```

## Evaluation signals

- feature_results/ensemble_RISE.npy exists in the job directory after run_feature completes ([INFO] Step2 Feature Done)
- Heatmap array shape matches expectations from input dimensions — demo log shows (6, 341, 341)
- show_feature with mode='ensemble' renders distinct saliency patterns per class (cancer/health/nodule) consistent with the ensemble's confusion-matrix performance (e.g., AUC 0.99 on the 859-sample serum test set)
- Salient regions identified on the CCLE lipidomics data (928 cell lines, 23 cancer types) yield metabolite-protein networks that recover the reported 14 shared metabolites and 3 shared proteins across all 23 networks
- Saliency extraction runs only on a completed ensemble training job; verify 10 DenseNet121 runs finished before run_feature

## Limitations

- The heatmap highlights spectral regions, not annotated metabolite identities; metabolite-protein association requires downstream PIUMet network mapping
- Pre-trained models (859 serum samples) are not distributed publicly — access requires contacting the author
- Retention of unknown/unannotated metabolite signals means salient regions may correspond to unidentified m/z features

## Evidence
