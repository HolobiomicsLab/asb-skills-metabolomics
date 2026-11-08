---
name: cross-ionization-mode-loss-evaluation
description: Use when you need to quantify whether dual-ionization-mode training trades
  off within-mode prediction accuracy, by benchmarking single-mode ablations on held-out
  same-mode spectrum pairs per Tanimoto bin.
when_to_use_negative:
- Your goal is cross-mode (positive-vs-negative) similarity prediction — single-mode
  ablation models cannot make cross-ionization-mode predictions at all
- Your dataset is small (< ~100,000 spectra); the README warns pair sampling can be
  suboptimal and the ablation comparison would be confounded by sampling artifacts
- You only need final similarity matrices from the pre-trained model rather than a
  training-strategy ablation
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network framework being ablated; single-mode variants trained
    with identical hyperparameters via its automatic training pipeline
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend in which the Siamese models are implemented and trained
- name: matchms
  role: Loading and filtering/cleaning the positive and negative reference spectra
    before training
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:1af82597895ca128f2bedcbafe6472e63381d0eac1b2ce07edce6a3f3c3cf648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-ionization-mode-loss-evaluation

## Summary

Trains single-ionization-mode Siamese models (positive-only and negative-only) with identical hyperparameters to the dual-mode MS2DeepScore 2.0 model, then compares within-mode losses per Tanimoto bin to isolate how joint training affects accuracy. This reproduces the Suppl. Fig 4.1-4.3 ablation that justifies (or questions) dual-mode training.

## When to use

Use when you need to quantify whether dual-ionization-mode training trades off within-mode prediction accuracy, by benchmarking single-mode ablations on held-out same-mode spectrum pairs per Tanimoto bin.

## When NOT to use

- Your goal is cross-mode (positive-vs-negative) similarity prediction — single-mode ablation models cannot make cross-ionization-mode predictions at all
- Your dataset is small (< ~100,000 spectra); the README warns pair sampling can be suboptimal and the ablation comparison would be confounded by sampling artifacts
- You only need final similarity matrices from the pre-trained model rather than a training-strategy ablation

## Inputs

- Positive ionization mode reference spectra from public spectral libraries (GNPS, MoNA, MassBank, MSnLib)
- Negative ionization mode reference spectra from the same libraries
- Dual-mode MS2DeepScore 2.0 model with its hyperparameters and per-Tanimoto-bin loss evaluation for baseline comparison
- matchms-cleaned spectra (mgf/msp/mzml formats) with Tanimoto labels from InChIKey/SMILES-derived structures

## Outputs

- model_comparison.csv containing per-Tanimoto-bin losses and summary metrics for positive-only, negative-only, and dual-mode models
- Within-mode loss-vs-Tanimoto-bin plots mirroring Suppl. Fig 4.1-4.3

## How to apply

Load positive and negative ionization mode reference spectra from the public spectral libraries (GNPS, MoNA, MassBank, MSnLib) and clean them with the matchms filtering pipeline. Using ms2deepscore's automatic training pipeline (SettingsMS2Deepscore + train_ms2deepscore_wrapper, PyTorch backend), train a positive-only model and a negative-only model with hyperparameters identical to the dual-mode model, including the new pair sampling algorithm that balances pairs over Tanimoto bins and molecule sampling frequency; set ionisation_mode='positive' or 'negative' in place of 'both'. Evaluate each single-mode model on held-out same-mode spectrum pairs and compute losses per Tanimoto bin, mirroring the evaluation protocol used for the dual-mode model. Record the comparison: positive-only within-mode losses should be comparable to the dual-mode model's positive-mode losses, while the negative-only model should show lower losses at low Tanimoto bins but higher loss in the 0.9-1.0 Tanimoto bin. Write the per-bin loss table and summary metrics to model_comparison.csv. Note the caveat that the new sampling algorithm did not enforce balanced sampling across ionization-mode pair types, which may explain residual differences in negative-vs-negative performance.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network framework being ablated; single-mode variants trained with identical hyperparameters via its automatic training pipeline) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Backend in which the Siamese models are implemented and trained)
- **matchms** (Loading and filtering/cleaning the positive and negative reference spectra before training) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; train_ms2deepscore_wrapper(SettingsMS2Deepscore(spectrum_file_path='positive_only.mgf', ionisation_mode='positive'), None)
```

## Evaluation signals

- Positive-only model's within-mode losses are comparable to the dual-mode model's positive-mode losses across Tanimoto bins
- Negative-only model shows lower losses than the dual-mode model for low Tanimoto bins
- Negative-only model shows higher loss than the dual-mode model in the 0.9-1.0 Tanimoto bin
- Per-bin loss table for each model covers the same Tanimoto bins and uses the same pair-sampling protocol as the dual-mode evaluation
- Single-mode training runs use identical hyperparameters to the dual-mode run (only ionisation_mode differs)

## Limitations

- The new pair sampling algorithm balances pairs over Tanimoto bins and molecule sampling frequency but does not enforce balanced sampling of ionization-mode pair types, possibly limiting negative-vs-negative performance of the dual-mode model
- Differences in Tanimoto score distributions between ionization modes may yield unequal numbers of pairs per Tanimoto bin per mode
- Training your own model is only recommended with > 100,000 sufficiently diverse spectra; for smaller datasets the pair sampling must be checked and re-optimized

## Evidence

- [results] Within-ionization-mode performance was comparable between positive-only and dual-mode models: [evidence span withheld — anchor status: not_found]
- [discussion] Sampling algorithm did not enforce balanced ionization mode pairs: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] Balanced ionization-mode sampling might improve negative-vs-negative predictions: "Having more balanced sampling over the ionization modes might improve model performance for negative vs. negative ionization mode predictions"
- [intro] New pair sampling balances pairs over Tanimoto bins: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [readme] README documents the training wrapper and ionisation_mode setting: [evidence span withheld — anchor status: external_doc]
