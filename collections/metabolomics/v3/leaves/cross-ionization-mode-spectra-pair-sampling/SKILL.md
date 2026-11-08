---
name: cross-ionization-mode-spectra-pair-sampling
description: Use when constructing training pairs for a Siamese spectral-similarity
  model (e.g. MS2DeepScore) from spectra spanning both ionization modes, especially
  when raw pair enumeration would over-represent high-frequency molecules or easy
  (very high/very low) Tanimoto bins.
when_to_use_negative:
- You are training a model on a single ionization mode only, e.g. reproducing a DreaMS-style
  positive-mode model — the cross-mode balancing logic is not needed
- 'The spectrum set is tiny (< ~100,000 spectra): the article/README recommend merging
  custom data with larger public libraries or fine-tuning the pre-trained model instead
  of retraining from scratch with re-optimized pair sampling'
- Pairs are needed only for evaluation on a held-out test set — sampling balance is
  a training-time bias-control step, not an evaluation requirement
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3334
tools:
- name: MS2DeepScore 2.0
  role: Model and training pipeline whose Siamese network consumes the sampled Tanimoto-stratified
    pairs; pair sampling is configured via SettingsMS2Deepscore with additional_metadata
    for ionmode and precursor m/z
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectrum cleaning pipeline applied before pair derivation to harmonize spectra
    from GNPS, MassBank EU, MoNA and MSnLib
  repo: https://github.com/matchms/matchms
- name: PyTorch
  role: Implementation framework for the Siamese network trained on the sampled pairs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:e41969354047b0d2b03394e29fdc996784bc51f90d790b8e854895b776f624d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-ionization-mode-spectra-pair-sampling

## Summary

A balanced pair-sampling algorithm for building Siamese-network training sets from tandem MS spectra in both positive and negative ionization modes. It stratifies spectrum pairs over Tanimoto similarity bins to reduce the sampling bias that would otherwise dominate MS2DeepScore 2.0 training.

## When to use

Use when constructing training pairs for a Siamese spectral-similarity model (e.g. MS2DeepScore) from spectra spanning both ionization modes, especially when raw pair enumeration would over-represent high-frequency molecules or easy (very high/very low) Tanimoto bins.

## When NOT to use

- You are training a model on a single ionization mode only, e.g. reproducing a DreaMS-style positive-mode model — the cross-mode balancing logic is not needed
- The spectrum set is tiny (< ~100,000 spectra): the article/README recommend merging custom data with larger public libraries or fine-tuning the pre-trained model instead of retraining from scratch with re-optimized pair sampling
- Pairs are needed only for evaluation on a held-out test set — sampling balance is a training-time bias-control step, not an evaluation requirement

## Inputs

- Preprocessed tandem MS spectra from GNPS, MassBank EU, MoNA and MSnLib libraries (mgf/msp etc.) with precursor m/z and ionmode metadata
- Known chemical structures (InChIKey/SMILES) per spectrum for computing ground-truth Tanimoto labels
- SettingsMS2Deepscore configuration (spectrum file path, additional_metadata, ionisation_mode='both')

## Outputs

- Stratified training/validation spectrum pairs with per-pair Tanimoto similarity targets and ionization-mode pairing type (pos-pos, neg-neg, cross-mode)
- Pair-sampling diagnostics (pairs per Tanimoto bin, molecule sampling frequency) for quality control

## How to apply

Start from cleaned spectra (matchms DEFAULT_FILTERS) that carry known structures, precursor m/z and ionmode metadata, then compute all-pairs Tanimoto similarities from InChIKey-derived structures. Instead of sampling pairs uniformly, stratify pairs over Tanimoto similarity bins (0–1 range) and balance the sampling frequency of individual molecules, so that each bin contributes a comparable number of training pairs. Supply ionization mode as a categorical (CategoricalToBinary: positive=1, negative=0) and standardized precursor m/z (mean 0, standard deviation 1000) as additional metadata alongside the binned fragment peaks; the article's new sampling algorithm deliberately did NOT enforce balanced sampling of ionization-mode pair types, so inspect the positive-positive, negative-negative, and cross-mode pair counts before training. The README's pair sampling tutorial must be checked and potentially re-optimized for new datasets, since pair sampling quality is dataset-dependent and particularly fragile for small training sets.

## Related tools

- **MS2DeepScore 2.0** (Model and training pipeline whose Siamese network consumes the sampled Tanimoto-stratified pairs; pair sampling is configured via SettingsMS2Deepscore with additional_metadata for ionmode and precursor m/z) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectrum cleaning pipeline applied before pair derivation to harmonize spectra from GNPS, MassBank EU, MoNA and MSnLib) — https://github.com/matchms/matchms
- **PyTorch** (Implementation framework for the Siamese network trained on the sampled pairs)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'}), ('StandardScaler', {'metadata_field': 'precursor_mz', 'mean': 0, 'standard_deviation': 1000})], ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Pair counts per Tanimoto bin should be approximately equal after stratification; the article warns of unequal bins when Tanimoto score distributions differ per ionization mode
- Ionization-mode pair-type composition (pos-pos, neg-neg, cross-mode) should be inspected for gross imbalance, since the algorithm does not enforce it
- Pair-sampling quality check from the pair_sampling_tutorial must pass (or be re-optimized) before launching train_ms2deepscore_wrapper
- Downstream validation: held-out cross-ionization-mode pairs show strong predicted-vs-true Tanimoto correlation (Pearson r, RMSE) consistent with Figure 2b
- Individual molecule sampling frequency should not be dominated by over-represented library entries

## Limitations

- The new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs; more balanced ionization-mode sampling might improve negative vs. negative mode performance
- Differences in Tanimoto score distributions between ionization modes can result in not having an equal number of pairs per Tanimoto bin for each ionization mode
- Particularly for smaller training sets, pair sampling can be suboptimal if not checked and re-optimized for the dataset
- Training with only annotated library spectra means MS1 data cannot be included, and adduct information is excluded from the default model because preprocessing tools do not reliably generate it

## Evidence

- [discussion] New pair sampling algorithm balances pairs over Tanimoto bins and molecule sampling frequency: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [intro] Motivation for the pair-sampling algorithm: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [readme] Pair sampling must be checked and re-optimized per dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] Bin imbalance caveat across ionization modes: "this might result in not having an equal number of pairs per Tanimoto bin for each ionization mode"
