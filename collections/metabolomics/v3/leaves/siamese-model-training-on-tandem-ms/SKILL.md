---
name: siamese-model-training-on-tandem-ms
description: Use when you have >100,000 cleaned tandem MS spectra (e.g. GNPS, MassBank,
  MoNA, MSnLib MGF) with structures, and need a learned spectral similarity model
  that spans both ionization modes.
when_to_use_negative:
- Your dataset has far fewer than ~100,000 spectra and you cannot merge with public
  libraries — pair sampling will likely be suboptimal; prefer fine-tuning the pre-trained
  model instead
- You only need to score spectra with an existing model (use the pre-trained ms2deepscore_model.pt
  rather than retraining)
- You need a cross-ionization-mode model built on top of a backbone trained only on
  positive ionization spectra (e.g. DreaMS is unsuitable for this purpose)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ms2deepscore
  role: Provides SettingsMS2Deepscore, SettingsEmbeddingEvaluator, and train_ms2deepscore_wrapper
    for the automatic Siamese model training pipeline (PyTorch backend)
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectrum cleaning/filtering pipeline applied before training; recommended
    preprocessing for spectral libraries
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend framework in which the entire MS2DeepScore 2.0 Siamese model is implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:628791b15d3895b3ab2a987e367d43c66fdc0eac3e6988ab47e7d86473644f7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# siamese-model-training-on-tandem-ms

## Summary

Train a Siamese neural network (MS2DeepScore 2.0) that predicts Tanimoto chemical similarity from pairs of tandem mass spectra, using balanced pair sampling over Tanimoto bins and optional metadata inputs (ionization mode, precursor m/z) so a single model works within and across positive/negative ionization modes.

## When to use

Use when you have >100,000 cleaned tandem MS spectra (e.g. GNPS, MassBank, MoNA, MSnLib MGF) with structures, and need a learned spectral similarity model that spans both ionization modes.

## When NOT to use

- Your dataset has far fewer than ~100,000 spectra and you cannot merge with public libraries — pair sampling will likely be suboptimal; prefer fine-tuning the pre-trained model instead
- You only need to score spectra with an existing model (use the pre-trained ms2deepscore_model.pt rather than retraining)
- You need a cross-ionization-mode model built on top of a backbone trained only on positive ionization spectra (e.g. DreaMS is unsuitable for this purpose)

## Inputs

- Cleaned tandem MS spectra with structures (mgf, msp, mzML, mzXML, json, USI) from public libraries such as GNPS, MoNA, MassBank, MSnLib (>100,000 spectra recommended)
- SettingsMS2Deepscore configuration (spectrum_file_path, additional_metadata, ionisation_mode, hyperparameters)
- SettingsEmbeddingEvaluator configuration (optional, to train the embedding evaluator)

## Outputs

- Trained MS2DeepScore 2.0 model file (e.g. ms2deepscore_model.pt) usable for same- and cross-ionization-mode similarity prediction
- Embedding evaluator model for per-spectrum prediction-quality estimation
- Spectrum embeddings (vectors) usable for UMAP visualization or molecular networking

## How to apply

First clean spectra with the matchms filtering pipeline, then assemble pairs with the new pair sampling algorithm that balances pairs over Tanimoto bins and molecule sampling frequency (check and potentially re-optimize sampling for your dataset, especially for small training sets). Configure SettingsMS2Deepscore with additional_metadata such as CategoricalToBinary on 'ionmode' (positive=1, negative=0) and StandardScaler on 'precursor_mz' (mean 0, sd 1000), and set ionisation_mode='both' to train one joint model rather than per-mode models. Run train_ms2deepscore_wrapper with SettingsEmbeddingEvaluator to also train the embedding evaluator that flags spectra with low information content or dissimilarity to training data. For small custom datasets, merge them with larger public libraries before training from scratch, or fine-tune starting from the pre-trained model. Note the sampling algorithm balances Tanimoto bins but does not enforce balanced sampling of ionization mode pairs, so evaluate within-mode losses per Tanimoto bin (e.g. comparing positive-only vs. dual-mode, negative-only vs. dual-mode) to detect mode-specific degradation.

## Related tools

- **ms2deepscore** (Provides SettingsMS2Deepscore, SettingsEmbeddingEvaluator, and train_ms2deepscore_wrapper for the automatic Siamese model training pipeline (PyTorch backend)) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectrum cleaning/filtering pipeline applied before training; recommended preprocessing for spectral libraries) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Backend framework in which the entire MS2DeepScore 2.0 Siamese model is implemented)

## Examples

```
settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'}), ('StandardScaler', {'metadata_field': 'precursor_mz', 'mean': 0, 'standard_deviation': 1000})], ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Within-ionization-mode losses per Tanimoto bin of the dual-mode model are comparable to a positive-only model trained with identical hyperparameters
- Pair sampling yields balanced numbers of pairs per Tanimoto bin (verify with the pair sampling tutorial; unequal pairs per bin signal sampling issues from Tanimoto score distribution differences)
- Embedding evaluator correctly flags spectra with low information content or spectra dissimilar to training data; removing high-predicted-uncertainty test spectra raises overall accuracy
- Model file loads with ms2deepscore.models.load_model and produces similarity matrices for both positive and negative as well as cross-ionization-mode spectrum pairs
- Metadata inputs are encoded as specified: ionmode as binary (positive=1, negative=0) and precursor_mz standardized with mean 0, sd 1000

## Limitations

- The new sampling algorithm balances pairs over Tanimoto bins but does not enforce balanced sampling of ionization mode pairs; more balanced mode sampling might improve negative-vs-negative performance
- Unequal numbers of pairs per Tanimoto bin per ionization mode can arise from differences in Tanimoto score distributions
- Fragment signals are initially binned, which might result in loss of useful information; MS1 data and adduct information are not included in the default model
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to training data
- Training your own model is only recommended with substantial data (>100,000 diverse spectra) and some machine learning familiarity

## Evidence

- [intro] we introduce a new pair sampling algorithm that reduces biases introduced during model training: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [intro] additional metadata improves performance: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [discussion] sampling balances Tanimoto bins but not ionization modes: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [readme] README training wrapper invocation: [evidence span withheld — anchor status: external_doc]
- [readme] README dataset size guidance: [evidence span withheld — anchor status: external_doc]
