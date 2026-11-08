---
name: tandem-ms-spectral-library-curation
description: Use when building a training or benchmark set for spectral similarity
  prediction from public MS/MS libraries, especially when spectra from both positive
  and negative ionization modes must be harmonized for cross-mode model training.
when_to_use_negative:
- 'Only a small custom dataset is available (< ~100,000 spectra): the README recommends
  merging it with the larger public libraries or fine-tuning the pre-trained model
  instead of curating/training from scratch.'
- 'You only need to score already-processed spectra with the released model: use the
  pre-trained ms2deepscore_model.pt from Zenodo directly rather than re-curating libraries.'
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3473
tools:
- name: matchms
  role: Spectral cleaning/filtering pipeline (DEFAULT_FILTERS) applied to library
    spectra before binning and pair generation
  repo: https://github.com/matchms/matchms
- name: ms2deepscore
  role: Consumes the curated, Tanimoto-labeled library spectra for Siamese network
    training via train_ms2deepscore_wrapper with SettingsMS2Deepscore
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:e489c5ba66945bb47eff0a07925d665737c1df6580bf3c26df2c00c914c49c88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tandem-ms-spectral-library-curation

## Summary

Curate public tandem MS spectral libraries (GNPS, MassBank EU, MoNA, MSnLib) into cleaned, binned, metadata-annotated training data for MS2DeepScore 2.0, attaching precursor m/z, ionization mode, and structure-derived Tanimoto similarity labels.

## When to use

Use when building a training or benchmark set for spectral similarity prediction from public MS/MS libraries, especially when spectra from both positive and negative ionization modes must be harmonized for cross-mode model training.

## When NOT to use

- Only a small custom dataset is available (< ~100,000 spectra): the README recommends merging it with the larger public libraries or fine-tuning the pre-trained model instead of curating/training from scratch.
- You only need to score already-processed spectra with the released model: use the pre-trained ms2deepscore_model.pt from Zenodo directly rather than re-curating libraries.

## Inputs

- Tandem MS spectra from GNPS, MassBank EU, MoNA, and MSnLib (mgf, msp, mzML, mzXML, json, or USI formats)
- Known chemical structures (e.g., SMILES/InChI) for library entries, used to compute Tanimoto similarity labels
- Spectral metadata: precursor m/z, ionization mode (positive/negative), adduct where available

## Outputs

- Cleaned, peak-binned spectra annotated with precursor m/z and ionmode metadata (e.g., combined_libraries.mgf)
- Training and held-out test split of spectra
- Tanimoto-labeled spectrum pairs stratified over Tanimoto similarity bins for Siamese network training

## How to apply

Retrieve spectra from the public libraries used in the paper (GNPS, MassBank EU, MoNA, MSnLib), then clean them with the matchms filtering pipeline (DEFAULT_FILTERS), since the README explicitly recommends 'using the cleaning pipeline in matchms' before training. Bin fragment peaks and attach precursor m/z and ionization mode metadata to each spectrum; for the MS2DeepScore 2.0 model, encode ionmode as a CategoricalToBinary feature and standardize precursor_mz (mean 0, standard deviation 1000). Derive ground-truth Tanimoto similarity labels from the known structures of the library entries and split spectra into training and held-out test sets. When training from scratch, generate pairs with the balanced pair-sampling algorithm that stratifies pairs over Tanimoto similarity bins, and verify pair-sampling quality per the pair sampling tutorial because it 'has to be checked and potentially re-optimized for new datasets'. For small custom datasets, merge them with the larger public libraries before training rather than training in isolation.

## Related tools

- **matchms** (Spectral cleaning/filtering pipeline (DEFAULT_FILTERS) applied to library spectra before binning and pair generation) — https://github.com/matchms/matchms
- **ms2deepscore** (Consumes the curated, Tanimoto-labeled library spectra for Siamese network training via train_ms2deepscore_wrapper with SettingsMS2Deepscore) — https://github.com/matchms/ms2deepscore

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; train_ms2deepscore_wrapper(SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'}), ('StandardScaler', {'metadata_field': 'precursor_mz', 'mean': 0, 'standard_deviation': 1000})], ionisation_mode='both'), None)
```

## Evaluation signals

- Every curated spectrum carries a valid ionmode annotation (positive/negative) and a precursor_mz field, standardizable with mean 0 and standard deviation 1000 as in SettingsMS2Deepscore
- Pair sampling output shows balanced pair counts across Tanimoto similarity bins; the pair sampling tutorial states sampling quality 'has to be checked and potentially re-optimized for new datasets'
- Held-out test spectra are disjoint from training spectra, with Tanimoto labels traceable to known structures
- Curated library size is in the same order as the released model's training data (> 500,000 MS/MS combined spectra from GNPS, MoNA, MassBank, MSnLib)
- Downstream check: held-out cross-ionization-mode predictions show strong predicted-vs-true Tanimoto correlation (Figure 2b reproduction, Pearson r and RMSE in correlation_metrics.csv)

## Limitations

- Fragment signals are binned, 'which might result in the loss of useful information'
- Public mass spectral libraries 'generally do not have annotated raw MS1 spectra available', so MS1 data cannot be included in training
- Adduct information is excluded from the default model because 'not all preprocessing tools generate reliable adduct information'
- Pair sampling may not be balanced across ionization modes or Tanimoto bins, 'which might result in not having an equal number of pairs per Tanimoto bin for each ionization mode'
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to the training data

## Evidence

- [readme] readme_training_data: [evidence span withheld — anchor status: not_found]
- [readme] readme_cleaning: [evidence span withheld — anchor status: not_found]
- [readme] readme_pair_sampling: [evidence span withheld — anchor status: external_doc]
- [discussion] article_merge_libraries: "For smaller custom datasets, we recommend merging them with larger available datasets, such as the here-used public libraries, before training a new model from scratch"
- [discussion] article_binning_loss: "Fragment signals are initially binned, which might result in the loss of useful information."
- [intro] article_metadata_input: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
