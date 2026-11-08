---
name: spectral-pair-sampling-across-tanimoto-bins
description: Use when training or fine-tuning an MS2DeepScore model (positive, negative,
  or dual ionization mode) and you need pair sampling that balances pairs over Tanimoto
  bins, or when comparing per-bin losses across models.
when_to_use_negative:
- Working with very small custom training sets without first checking/optimizing pair
  sampling quality, since pair sampling can be suboptimal if not verified
- Training a cross-ionization mode model on a backbone like DreaMS that has only been
  trained on positive ionization spectra
- Predicting similarity for spectra with unreliable adduct information you plan to
  add as metadata input
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese model trained with the balanced Tanimoto-bin pair sampling algorithm;
    provides the automatic training pipeline in ms2deepscore
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend implementing the Siamese network trained on the sampled spectrum pairs
- name: matchms
  role: Cleaning/preprocessing of spectra before pair sampling and training
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:38eb6ea18431ca8b4768003cab057dd385ed2d30faed906cc2f806fdbc7ba2a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-pair-sampling-across-tanimoto-bins

## Summary

A training-data construction skill for MS2DeepScore 2.0-style Siamese models that samples spectrum pairs balanced over Tanimoto similarity bins and molecule sampling frequency, reducing training biases and enabling fair per-bin loss evaluation.

## When to use

Use when training or fine-tuning an MS2DeepScore model (positive, negative, or dual ionization mode) and you need pair sampling that balances pairs over Tanimoto bins, or when comparing per-bin losses across models.

## When NOT to use

- Working with very small custom training sets without first checking/optimizing pair sampling quality, since pair sampling can be suboptimal if not verified
- Training a cross-ionization mode model on a backbone like DreaMS that has only been trained on positive ionization spectra
- Predicting similarity for spectra with unreliable adduct information you plan to add as metadata input

## Inputs

- Cleaned mass spectra (mgf, msp, mzML, mzXML, json, usi) from public libraries such as GNPS, MoNA, MassBank, MSnLib
- Tanimoto similarity scores computed from structure annotations (SMILES/InChI) of the reference spectra
- SettingsMS2Deepscore configuration (e.g., ionisation_mode, additional_metadata like ionmode and precursor_mz)

## Outputs

- Balanced training/validation spectrum pairs distributed over Tanimoto bins
- Per-Tanimoto-bin loss table (e.g., model_comparison.csv) comparing single-mode vs. dual-mode models

## How to apply

Load cleaned reference spectra (e.g., GNPS, MoNA, MassBank, MSnLib merged libraries) and compute Tanimoto structural similarities for candidate pairs, then bin the pairs by Tanimoto score. Use the new pair sampling algorithm from ms2deepscore to balance the number of pairs per Tanimoto bin and the molecule sampling frequency, while checking that spectra are sufficiently cleaned via the matchms cleaning pipeline first. Note the algorithm balances Tanimoto bins and molecule frequency but does NOT enforce balanced sampling of ionization-mode pairs, so equal pairs per bin per ionization mode is not guaranteed when Tanimoto distributions differ. After training, evaluate losses per Tanimoto bin on held-out same-mode pairs; validation includes confirming comparable within-mode losses for positive-only vs. dual-mode models, and that negative-only losses are lower at low Tanimoto bins but higher in the 0.9-1.0 bin.

## Related tools

- **MS2DeepScore 2.0** (Siamese model trained with the balanced Tanimoto-bin pair sampling algorithm; provides the automatic training pipeline in ms2deepscore) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Backend implementing the Siamese network trained on the sampled spectrum pairs)
- **matchms** (Cleaning/preprocessing of spectra before pair sampling and training) — https://github.com/matchms/matchms

## Examples

```
settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Pair counts per Tanimoto bin are approximately balanced within the training set (algorithm balances pairs over Tanimoto bins and molecule sampling frequency)
- Within-ionization-mode losses for positive-only and dual-mode models are comparable on held-out same-mode pairs
- Negative-only model losses are lower than the dual-mode model at low Tanimoto bins but higher in the 0.9-1.0 Tanimoto bin
- Pair sampling quality was checked and, if needed, re-optimized for the specific dataset (recommended in the pair sampling tutorial)
- Per-bin loss table and summary metrics written to model_comparison.csv validate against the dual-mode evaluation

## Limitations

- The new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs; more balanced ionization-mode sampling might improve negative vs. negative mode performance
- Differences in Tanimoto score distributions between ionization modes may result in not having an equal number of pairs per Tanimoto bin for each ionization mode
- For smaller training sets, pair sampling can be suboptimal if not checked and potentially re-optimized

## Evidence

- [intro] Introduce a new pair sampling algorithm that reduces biases introduced during model training: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [discussion] Pair sampling balanced over Tanimoto bins but not ionization modes: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] Unequal pairs per bin per mode: "this might result in not having an equal number of pairs per Tanimoto bin for each ionization mode"
- [readme] Check pair sampling quality for new datasets: [evidence span withheld — anchor status: external_doc]
- [readme] Training wrapper usage with settings: [evidence span withheld — anchor status: external_doc]
