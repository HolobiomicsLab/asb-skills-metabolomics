---
name: spectral-data-augmentation-gaussian-noise
description: Use when training a deep learning model on MS/MS spectra from an instrument
  type with limited training data (e.g., Q-TOF) and top-1 formula prediction accuracy
  is near zero without augmentation.
when_to_use_negative:
- Training the Orbitrap model — the article states no data augmentation was applied
  there because noise addition led to lower spectral similarity and the Orbitrap training
  set is already large
- The instrument type has abundant training data, so augmentation adds variance without
  benefit
- Evaluation on the held-out test split — augmentation is a training-time technique
  and must not perturb test spectra
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN training pipeline (train_tcn_gpus.py) consumes
    the Gaussian-noise-augmented Q-TOF spectra
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type whose limited training data motivates the augmentation; noise
    (mean 0, sd 0.1) is applied to Q-TOF spectrum intensities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:9db9cdd3fb2a9f0604f6578ad6eb2d8c111e1128732f503329c2deb1ff81bfa4
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-data-augmentation-gaussian-noise

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augments MS/MS training spectra by adding Gaussian noise (mean 0, standard deviation 0.1) to spectrum intensities, as used to expand limited Q-TOF training data when training FIDDLE's TCN formula-prediction model. This augmentation raised correctly predicted top-1 formulas from 0% to 9% (all atoms) and 18% to 26% (heavy atoms) in the ablation study.

## When to use

Use when training a deep learning model on MS/MS spectra from an instrument type with limited training data (e.g., Q-TOF) and top-1 formula prediction accuracy is near zero without augmentation.

## When NOT to use

- Training the Orbitrap model — the article states no data augmentation was applied there because noise addition led to lower spectral similarity and the Orbitrap training set is already large
- The instrument type has abundant training data, so augmentation adds variance without benefit
- Evaluation on the held-out test split — augmentation is a training-time technique and must not perturb test spectra

## Inputs

- MoNA Q-TOF MS/MS spectra converted to MGF format (title, precursor_mz, precursor_type, collision_energy fields)
- Q-TOF TCN training configuration files from ./config/
- Preprocessed spectra passing the standard filters (>=5 peaks, m/z in (0, 1500], peaks below 50 m/z removed)

## Outputs

- Augmented Q-TOF training set with Gaussian-noise-perturbed spectrum intensities
- Trained TCN model checkpoint (e.g., fiddle_tcn_qtof.pt) with improved top-1 formula prediction
- Ablation comparison of missed-atom histograms ('Pred' vs 'Pred w/o DA')

## How to apply

During training of the FIDDLE TCN predictor (train_tcn_gpus.py with the ./config/ Q-TOF config), apply random noise sampled from a Gaussian distribution with mean 0 and standard deviation 0.1 to the spectrum intensities only — precursor m/z and other features stay unmodified. This simulates instrument variability to expand the effective Q-TOF training set. To quantify the contribution, train an ablation variant 'Pred w/o DA' with the augmentation disabled and compare missed-atom distributions of top-1 predictions against the full 'Pred' model. Critically, do not apply this augmentation when the instrument type is Orbitrap: the article reports that adding noise lowered similarity for Orbitrap spectra, which already have a sufficiently large training set.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN training pipeline (train_tcn_gpus.py) consumes the Gaussian-noise-augmented Q-TOF spectra) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type whose limited training data motivates the augmentation; noise (mean 0, sd 0.1) is applied to Q-TOF spectrum intensities)

## Examples

```
python train_tcn_gpus.py --config ./config/qtof.yaml  # full 'Pred' variant with Gaussian noise (mean 0, sd 0.1) on spectrum intensities; omit noise for the 'Pred w/o DA' ablation
```

## Evaluation signals

- Ablation delta: correctly predicted top-1 formulas should rise from 0% to 9% (all atoms) and from 18% to 26% (heavy atoms) on the MoNA Q-TOF test split when augmentation is enabled
- Missed-atom histograms for the full 'Pred' model should show fewer missed atoms than the 'Pred w/o DA' variant
- Augmented spectra must retain original precursor m/z and precursor_type; only intensities are perturbed
- Noise parameters must match mean 0 and standard deviation 0.1 exactly to reproduce the paper's configuration

## Limitations

- No data augmentation was applied for Orbitrap training — noise addition led to lower similarity for Orbitrap spectra
- The benefit is instrument-specific; accuracy still varies across experimental conditions with less training data
- Augmentation alone does not capture all performance: contrastive loss and post-processing contribute further gains in the ablation

## Evidence

- [results] Data augmentation increases correctly predicted top-1 formulas from 0% to 9% (all atoms) and 18% to 26% (heavy atoms): [evidence span withheld — anchor status: not_found]
