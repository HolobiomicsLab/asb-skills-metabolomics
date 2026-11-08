---
name: tanimoto-similarity-bin-comparison
description: Use when comparing prediction accuracy of similarity models across Tanimoto
  bins — e.g. checking whether a dual-ionization-mode model matches single-mode baselines
  within each mode, or locating which Tanimoto bin (low vs 0.9–1.0) drives performance
  differences.
when_to_use_negative:
- Spectra lack annotated structures, so no ground-truth Tanimoto scores exist to bin
  predictions against
- The goal is cross-ionization-mode prediction quality rather than within-mode baseline
  comparison
- The custom dataset is too small for training from scratch (<~100,000 spectra) —
  fine-tune the pre-trained model instead of running single- vs dual-mode training
  comparisons
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network model whose single-mode (positive-only, negative-only)
    and dual-mode variants are trained and compared per Tanimoto bin
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend implementing the Siamese model used for all training arms of the comparison
- name: matchms
  role: Loading and cleaning the positive and negative mode reference spectra from
    public spectral libraries before training
  repo: https://github.com/matchms/matchms
- name: ms2deepscore
  role: Automatic training pipeline (train_ms2deepscore_wrapper, SettingsMS2Deepscore)
    and pair sampling algorithm balancing pairs over Tanimoto bins
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:e949e3cf71b7de8a689d7166ba774af336c58fba0595859d5b1e206cb702f0b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tanimoto-similarity-bin-comparison

## Summary

A benchmarking pattern that evaluates spectral-similarity models per Tanimoto similarity bin rather than only on aggregate loss, used here to compare single-ionization-mode (positive-only, negative-only) Siamese models against the dual-mode MS2DeepScore 2.0 model with identical hyperparameters.

## When to use

Use when comparing prediction accuracy of similarity models across Tanimoto bins — e.g. checking whether a dual-ionization-mode model matches single-mode baselines within each mode, or locating which Tanimoto bin (low vs 0.9–1.0) drives performance differences.

## When NOT to use

- Spectra lack annotated structures, so no ground-truth Tanimoto scores exist to bin predictions against
- The goal is cross-ionization-mode prediction quality rather than within-mode baseline comparison
- The custom dataset is too small for training from scratch (<~100,000 spectra) — fine-tune the pre-trained model instead of running single- vs dual-mode training comparisons

## Inputs

- Positive and negative ionization mode reference tandem MS spectra from public spectral libraries (MGF/MSP/JSON, e.g. the >500,000 combined GNPS/MoNA/MassBank/MSnLib spectra)
- Known structures (InChIKeys) for reference spectra to compute ground-truth Tanimoto scores
- Trained dual-mode MS2DeepScore 2.0 model checkpoint (ms2deepscore_model.pt) as the reference against single-mode retrainings
- Identical SettingsMS2Deepscore hyperparameters across all compared models

## Outputs

- Per-Tanimoto-bin loss table and summary metrics (model_comparison.csv)
- Within-mode comparison: positive-only vs dual-mode positive losses, negative-only vs dual-mode negative losses

## How to apply

Load the positive and negative ionization mode reference spectra from public spectral libraries (GNPS, MoNA, MassBank, MSnLib) using matchms and clean them with the matchms filtering pipeline. Train a positive-only Siamese model and a negative-only model with hyperparameters identical to the dual-mode MS2DeepScore 2.0 model, using the automatic training pipeline in ms2deepscore (PyTorch backend) and the new pair sampling algorithm that balances pairs over Tanimoto bins. Evaluate both models on held-out same-mode spectrum pairs, stratifying loss by Tanimoto bin, and record where the negative-only model shows lower losses at low Tanimoto bins but higher loss in the 0.9–1.0 bin versus the dual-mode model. Write the per-bin loss table and summary metrics to model_comparison.csv; the comparison isolates whether joint cross-mode training degrades within-mode accuracy.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network model whose single-mode (positive-only, negative-only) and dual-mode variants are trained and compared per Tanimoto bin) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Backend implementing the Siamese model used for all training arms of the comparison)
- **matchms** (Loading and cleaning the positive and negative mode reference spectra from public spectral libraries before training) — https://github.com/matchms/matchms
- **ms2deepscore** (Automatic training pipeline (train_ms2deepscore_wrapper, SettingsMS2Deepscore) and pair sampling algorithm balancing pairs over Tanimoto bins) — https://github.com/matchms/ms2deepscore

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore
settings = SettingsMS2Deepscore(spectrum_file_path="./combined_libraries.mgf", additional_metadata=[("CategoricalToBinary", {"metadata_field": "ionmode", "entries_becoming_one": "positive", "entries_becoming_zero": "negative"}), ("StandardScaler", {"metadata_field": "precursor_mz", "mean": 0, "standard_deviation": 1000})], ionisation_mode="both")
```

## Evaluation signals

- Positive-only within-mode losses are comparable to the dual-mode model's positive-mode losses
- Negative-only losses are lower than dual-mode at low Tanimoto bins but higher in the 0.9–1.0 Tanimoto bin
- Pairs per Tanimoto bin are balanced by the new pair sampling algorithm for each training arm
- All three models use identical hyperparameters (SettingsMS2Deepscore) so differences attribute to ionization-mode training data
- Per-bin loss table written to model_comparison.csv with one row per Tanimoto bin per model

## Limitations

- The new pair sampling algorithm balances pairs over Tanimoto bins and molecule sampling frequency but does not enforce balanced sampling of ionization-mode pairs, so unequal pairs per Tanimoto bin per mode (due to differing Tanimoto score distributions) may limit negative vs negative mode performance
- Training from scratch requires a substantial dataset (say > 100,000 spectra of sufficiently diverse types); pair sampling must be checked and potentially re-optimized, particularly for smaller training sets
- Unequal numbers of pairs per Tanimoto bin per ionization mode can bias the per-bin comparison

## Evidence

- [discussion] Unbalanced ionization-mode pair sampling limits negative-negative performance: "Having more balanced sampling over the ionization modes might improve model performance for negative vs. negative ionization mode predictions"
- [discussion] Pair sampling balances Tanimoto bins but not ionization-mode pairs: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [supplementary] Per-bin comparison rationale from workflow: [evidence span withheld — anchor status: not_found]
- [readme] Training pipeline invocation from README: [evidence span withheld — anchor status: external_doc]
- [readme] Reference library scope from README: [evidence span withheld — anchor status: external_doc]
