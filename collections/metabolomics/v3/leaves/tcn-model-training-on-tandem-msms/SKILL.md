---
name: tcn-model-training-on-tandem-msms
description: Use when training a formula-prediction deep learning model from matched
  MS/MS spectral libraries (e.g., MoNA Q-TOF, NIST23 Orbitrap) or when reproducing
  the paper's cross-instrument generalization experiment (Fig. 4j).
when_to_use_negative:
- The goal is formula inference on new spectra with existing weights — use pre-trained
  checkpoints or the msfiddle CLI instead of retraining
- Spectra fall outside the paper's filter space (fewer than five peaks, m/z above
  1500, non-frequent atoms, >300 atoms, or uncommon precursor types)
- The task is structural elucidation or fingerprint prediction (SIRIUS/CSI:FingerID
  territory), not molecular formula prediction
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN model (stacked dilated convolutions with
    large kernels) is trained on Q-TOF and Orbitrap MS/MS spectra via train_tcn_gpus.py
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type providing lower-resolution training spectra; noise augmentation
    with Gaussian noise (mean 0, sd 0.1) is optional
- name: Orbitrap
  role: Instrument type providing higher-resolution training spectra; trained without
    data augmentation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:1cb17e57fd5dfd8e403d761907a8a4fdc93dec6fd051f29871ea93dd4aeca34e
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
  - Orbitrap
  - Q-TOF
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tcn-model-training-on-tandem-msms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train FIDDLE's temporal convolutional network (TCN) models to predict molecular formulas from tandem MS/MS spectra, separately for Q-TOF and Orbitrap instrument types using the FIDDLE repository's train_tcn_gpus.py with instrument-specific YAML configs.

## When to use

Use when training a formula-prediction deep learning model from matched MS/MS spectral libraries (e.g., MoNA Q-TOF, NIST23 Orbitrap) or when reproducing the paper's cross-instrument generalization experiment (Fig. 4j).

## When NOT to use

- The goal is formula inference on new spectra with existing weights — use pre-trained checkpoints or the msfiddle CLI instead of retraining
- Spectra fall outside the paper's filter space (fewer than five peaks, m/z above 1500, non-frequent atoms, >300 atoms, or uncommon precursor types)
- The task is structural elucidation or fingerprint prediction (SIRIUS/CSI:FingerID territory), not molecular formula prediction

## Inputs

- Filtered MS/MS spectra from MoNA (Q-TOF) and NIST23 (Orbitrap, downsampled) linked into matched pairs by compound
- Instrument-specific YAML training configs (e.g., config/fiddle_tcn_orbitrap.yml)
- Molecule annotations with canonical SMILES and precursor types ([M + H]+, [M + Na]+, [2M + H]+, etc.)

## Outputs

- Trained TCN model checkpoints (one per instrument type)
- Missed-atom counts per predicted formula on test spectra (2x2 same- vs cross-instrument matrix)
- Figures such as fig4j_missed_atom_matrix.png

## How to apply

First compile filtered training spectra: keep spectra with at least five peaks, m/z in (0, 1500], molecules composed only of frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), total atom count capped at 300, and common precursor types such as [M + H]+. Split training and test sets by molecular canonical SMILES so no compound overlaps. For Q-TOF training, optionally augment by adding Gaussian noise (mean 0, sd 0.1) to spectrum intensities; for Orbitrap training, skip augmentation since noise addition lowered similarity for high-resolution spectra. Train with train_tcn_gpus.py using the instrument-specific config (e.g., config/fiddle_tcn_orbitrap.yml for Orbitrap, the Q-TOF config for Q-TOF), then evaluate checkpoints by recording missed-atom counts per predicted formula on matched test spectra.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN model (stacked dilated convolutions with large kernels) is trained on Q-TOF and Orbitrap MS/MS spectra via train_tcn_gpus.py) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type providing lower-resolution training spectra; noise augmentation with Gaussian noise (mean 0, sd 0.1) is optional)
- **Orbitrap** (Instrument type providing higher-resolution training spectra; trained without data augmentation)

## Examples

```
python train_tcn_gpus.py --config config/fiddle_tcn_orbitrap.yml
```

## Evaluation signals

- The 2x2 missed-atom matrix should show the Fig. 4j pattern: the Orbitrap-trained model incurs fewer missed atoms than the Q-TOF-trained model when tested on either instrument type
- Training and test sets share no common compounds when split by molecular canonical SMILES
- Trained models should be consistent with the paper's reported top-1 accuracy of 88.3% and top-5 accuracy of 93.6% on the evaluation set
- All training spectra satisfy the filters: ≥5 peaks, m/z ≤ 1500, frequent atoms only, ≤300 atoms per molecule

## Limitations

- Orbitrap training must not use Gaussian noise augmentation; the paper reports it led to lower similarity for Orbitrap spectra
- Accuracy varies across experimental conditions, especially those with less training data
- The Q-TOF model underperforms the Orbitrap model even on Q-TOF data due to lower spectral resolution of its training set

## Evidence
