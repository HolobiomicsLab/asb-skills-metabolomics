---
name: deep-learning-model-training-gpu
description: 'Use when reproducing or extending FIDDLE''s formula-identification training:
  you need to train TCN spectrum-prediction variants and a rescore model on MoNA/NIST
  Q-TOF MS/MS spectra with GPU acceleration and compare ablation contributions.'
when_to_use_negative:
- 'Training on Orbitrap data: FIDDLE did not apply Gaussian-noise data augmentation
  there because it lowered spectral similarity, and the Orbitrap training set is already
  large'
- You only need inference on pre-converted spectra — use the pre-trained checkpoints
  or the msfiddle PyPI package instead of retraining
- Spectra fail the preprocessing filters (fewer than five peaks, m/z above 1500, rare
  precursor types, atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN prediction models and rescore model are
    trained on GPUs via train_tcn_gpus.py, train_tcn_gpus_cl.py, and train_rescore.py
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument class whose Q-TOF MoNA spectra serve as the training data with
    the 10 ppm precursor mass tolerance and Gaussian-noise augmentation
- name: msfiddle
  role: PyPI package exposing FIDDLE models and msfiddle-download-models for checkpoint
    retrieval during setup
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:57d88fc95055b2cdfbf716b255e192b22cca3a9a87138887567068151b47ab99
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Q-TOF
  - msfiddle
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# deep-learning-model-training-gpu

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train deep learning models on GPU(s) for chemical formula prediction from tandem MS/MS spectra, following FIDDLE's TCN ablation setup (full model, no data augmentation, no contrastive loss) plus the secondary confidence-scoring (rescore) model on Q-TOF data.

## When to use

Use when reproducing or extending FIDDLE's formula-identification training: you need to train TCN spectrum-prediction variants and a rescore model on MoNA/NIST Q-TOF MS/MS spectra with GPU acceleration and compare ablation contributions.

## When NOT to use

- Training on Orbitrap data: FIDDLE did not apply Gaussian-noise data augmentation there because it lowered spectral similarity, and the Orbitrap training set is already large
- You only need inference on pre-converted spectra — use the pre-trained checkpoints or the msfiddle PyPI package instead of retraining
- Spectra fail the preprocessing filters (fewer than five peaks, m/z above 1500, rare precursor types, atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K)

## Inputs

- MoNA/NIST Q-TOF MS/MS spectra converted to MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- ./config/ Q-TOF configuration files for the three TCN ablation variants
- Pre-trained weights fiddle_tcn_qtof.pt and fiddle_rescore_qtof.pt (for reference/finetuning setup)
- environment.yml conda specification

## Outputs

- Trained TCN model checkpoints for 'Pred', 'Pred w/o DA', and 'Pred w/o CL' variants
- Trained secondary confidence-scoring (rescore) model checkpoints per variant
- Models enabling top-1 formula prediction and confidence-score ranking of candidate formulas

## How to apply

Create the conda environment from environment.yml, then fetch the Q-TOF pre-trained weights (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) via running_scripts/download_models.sh. Convert MS/MS spectra to MGF format with title, precursor_mz, precursor_type, and collision_energy fields, applying the standard filters (>=5 peaks, m/z in (0, 1500], peaks below 50 m/z removed, 10 ppm precursor mass tolerance for Q-TOF). Train the three TCN ablation variants with train_tcn_gpus.py (full 'Pred' with Gaussian noise augmentation, mean 0 and sd 0.1, on intensities only) and train_tcn_gpus_cl.py (contrastive-learning objective) using the ./config/ Q-TOF config files; note Gaussian-noise augmentation is Q-TOF-specific and was not applied for Orbitrap training. Then train the secondary confidence-scoring model for each variant with train_rescore.py. Datasets are split by molecular canonical SMILES to prevent compound leakage between training and test sets.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN prediction models and rescore model are trained on GPUs via train_tcn_gpus.py, train_tcn_gpus_cl.py, and train_rescore.py) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument class whose Q-TOF MoNA spectra serve as the training data with the 10 ppm precursor mass tolerance and Gaussian-noise augmentation)
- **msfiddle** (PyPI package exposing FIDDLE models and msfiddle-download-models for checkpoint retrieval during setup) — https://github.com/josiehong/msfiddle

## Examples

```
conda env create -f environment.yml; bash running_scripts/download_models.sh; python train_tcn_gpus.py --config ./config/qtof_pred.yaml; python train_rescore.py --config ./config/qtof_rescore.yaml
```

## Evaluation signals

- With data augmentation, correctly predicted top-1 formulas rise from 0% to 9% (all atoms) and 18% to 26% (heavy atoms) on the Q-TOF ablation; contrastive loss further improves over the no-CL variant
- Trained models + post-processing achieve top-5 candidate coverage of the correct formula above 84% on the held-out Q-TOF test split
- Confidence scores from the rescore model reach an ROC AUC of ~0.97 distinguishing correct from incorrect formulas
- Training/test split shows no common compounds (split by molecular canonical SMILES)
- Full 'Pred' model matches Fig. 4a-d missed-atom histogram patterns vs. the ablation variants

## Limitations

- Gaussian-noise data augmentation (mean 0, sd 0.1) is specific to Q-TOF; applying it to Orbitrap training lowered similarity and was omitted
- Experimental condition effects are not fully eliminated; accuracy varies across conditions, especially those with less training data
- v2.0.0 redesigns the rescore model (Siamese architecture), so checkpoints across versions are incompatible (see CHANGELOG.md)

## Evidence
