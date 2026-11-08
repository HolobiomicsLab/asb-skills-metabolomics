---
name: siamese-spectra-embedding-training
description: Use when you have >100,000 cleaned tandem mass spectra from both positive
  and negative ionization modes with reference structures, and need a model whose
  embeddings predict Tanimoto similarity for cross-ionization molecular networking.
when_to_use_negative:
- Your training set is small (fewer than ~100,000 spectra) and you have not checked/re-optimized
  pair sampling — the article recommends merging with larger public libraries or fine-tuning
  the pre-trained model instead.
- You need reliable adduct type as model input but your preprocessing tool does not
  generate trustworthy adduct information; the default model deliberately excludes
  adduct input for this reason.
- You want to train a cross-ionization model from a base model pretrained only on
  positive ionization data (e.g. DreaMS), which the article found unsuitable for cross-ionization
  training.
- Your spectra are mostly raw MS1-only acquisitions — MS1 data was excluded from training
  because public spectral libraries generally lack annotated raw MS1 spectra.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: MS2DeepScore 2.0
  role: The Siamese neural network framework being trained; predicts Tanimoto similarity
    between mass spectra pairs and works across ionization modes
  repo: github.com/matchms/MS2DeepScore
- name: ms2deepscore
  role: Python library providing SettingsMS2Deepscore, SettingsEmbeddingEvaluator,
    train_ms2deepscore_wrapper, and the MS2DeepScore scoring class for data preparation,
    training, and similarity computation
  repo: github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend framework in which the Siamese model is implemented and trained
- name: matchms
  role: Spectrum cleaning/filtering pipeline applied to spectra before training (DEFAULT_FILTERS
    via the matchms Pipeline)
  repo: https://github.com/matchms/matchms
- name: UMAP
  role: Dimensionality reduction for visualizing the learned spectrum embeddings in
    chemical space
- name: mzmine
  role: Local GUI deployment of the trained MS2DeepScore model for molecular networking
    without programming expertise
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:97366321cf93c751ae37c8deea86f4817939c96cc84ba424e01bfe5c27f2d2ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - SettingsMS2Deepscore
  - UMAP
  - matchms
  - ms2deepscore
  - mzmine
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6896551724137931
n_claims: 29
---

# siamese-spectra-embedding-training

## Summary

Train an MS2DeepScore 2.0 Siamese neural network that maps tandem mass spectra (positive and negative ionization modes) to embeddings whose Euclidean distances predict Tanimoto chemical similarity, using a balanced pair-sampling algorithm and optional metadata inputs (precursor m/z, ionization mode) to shape the embedding space.

## When to use

Use when you have >100,000 cleaned tandem mass spectra from both positive and negative ionization modes with reference structures, and need a model whose embeddings predict Tanimoto similarity for cross-ionization molecular networking.

## When NOT to use

- Your training set is small (fewer than ~100,000 spectra) and you have not checked/re-optimized pair sampling — the article recommends merging with larger public libraries or fine-tuning the pre-trained model instead.
- You need reliable adduct type as model input but your preprocessing tool does not generate trustworthy adduct information; the default model deliberately excludes adduct input for this reason.
- You want to train a cross-ionization model from a base model pretrained only on positive ionization data (e.g. DreaMS), which the article found unsuitable for cross-ionization training.
- Your spectra are mostly raw MS1-only acquisitions — MS1 data was excluded from training because public spectral libraries generally lack annotated raw MS1 spectra.

## Inputs

- Combined tandem mass spectral library in .mgf (or other common formats: msp, mzml, mzxml, json, usi) with spectra from both ionization modes
- Per-spectrum metadata: precursor m/z, ionmode, (optionally adduct type, instrument type)
- Reference chemical structures (e.g. SMILES) to compute Tanimoto similarity training labels
- SettingsMS2Deepscore configuration (spectrum_file_path, additional_metadata, ionisation_mode)
- SettingsEmbeddingEvaluator configuration (optional, to also train the embedding evaluator)

## Outputs

- Trained Siamese neural network checkpoint (ms2deepscore_model.pt)
- Embedding evaluator model that predicts per-spectrum prediction uncertainty
- Predicted Tanimoto similarity scores between spectrum pairs
- Spectrum embeddings usable for UMAP visualization of chemical space
- Similarity matrix (numpy array) over all input spectra

## How to apply

Clean the spectra with the matchms filtering pipeline, then configure training through SettingsMS2Deepscore, pointing spectrum_file_path at a combined .mgf of the libraries and setting ionisation_mode='both' so the Siamese network learns to embed spectra from both polarities in one space. Define additional_metadata conversions — a CategoricalToBinary encoder mapping ionmode 'positive'→1 / 'negative'→0 and a StandardScaler on precursor_mz (mean 0, standard_deviation 1000) — because the article shows precursor m/z and ionization mode as input improve prediction quality while one-hot instrument type does not. The pair sampling algorithm balances spectrum pairs over Tanimoto similarity bins and molecule sampling frequency, and its quality must be checked (and potentially re-optimized) via the pair sampling tutorial before training, especially for smaller training sets. Train the Siamese network in PyTorch, optionally passing SettingsEmbeddingEvaluator to train_ms2deepscore_wrapper so the model also trains an embedding evaluator that flags spectra with low information content or embeddings dissimilar to the training data. After training, judge success by RMSE between predicted and reference Tanimoto scores on held-out pairs, confirming the metadata-inclusive model outperforms the no-metadata baseline.

## Related tools

- **MS2DeepScore 2.0** (The Siamese neural network framework being trained; predicts Tanimoto similarity between mass spectra pairs and works across ionization modes) — github.com/matchms/MS2DeepScore
- **ms2deepscore** (Python library providing SettingsMS2Deepscore, SettingsEmbeddingEvaluator, train_ms2deepscore_wrapper, and the MS2DeepScore scoring class for data preparation, training, and similarity computation) — github.com/matchms/ms2deepscore
- **PyTorch** (Backend framework in which the Siamese model is implemented and trained)
- **matchms** (Spectrum cleaning/filtering pipeline applied to spectra before training (DEFAULT_FILTERS via the matchms Pipeline)) — https://github.com/matchms/matchms
- **UMAP** (Dimensionality reduction for visualizing the learned spectrum embeddings in chemical space)
- **mzmine** (Local GUI deployment of the trained MS2DeepScore model for molecular networking without programming expertise)

## Examples

```
settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'}), ('StandardScaler', {'metadata_field': 'precursor_mz', 'mean': 0, 'standard_deviation': 1000})], ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- RMSE between predicted and reference Tanimoto chemical similarities on held-out spectrum pairs improves over the no-metadata baseline when precursor m/z and ionmode metadata are included as input.
- Embedding quality is monitored by the embedding evaluator, which should flag spectra with low information content or embeddings far from the training-data distribution.
- Pair sampling distribution is balanced across Tanimoto similarity bins and ionization-mode pair types before and after training (verify via the pair sampling tutorial plots).
- The trained model produces similarity predictions valid for positive–positive, negative–negative, and cross-ionization-mode spectrum pairs in one embedding space.
- The model checkpoint loads via ms2deepscore.models.load_model and reproduces reasonable similarity scores on the pesticides_processed.mgf test resource.

## Limitations

- Pair sampling did not enforce balanced sampling of the different ionization mode pairs, which might limit negative vs. negative mode prediction performance.
- Tanimoto score distributions differ across ionization modes, potentially yielding unequal numbers of pairs per Tanimoto bin per mode.
- Adduct information was excluded from the default model because not all preprocessing tools generate reliable adduct annotations; incorrect adduct input could reduce performance.
- Fragment signals are initially binned, which might result in the loss of useful information.
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra from multiple precursor ions, or spectra dissimilar to the training data; there is also a risk the model is not well-suited for very specific tasks or chemical classes.
- Training your own model is only recommended if you have familiarity with machine learning and a substantial, diverse training set (> 100,000 spectra).

## Evidence

- [intro] Pair sampling balances over Tanimoto bins: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [intro] Metadata improves prediction: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [intro] Cross-ionization mode model: "Here, we developed a mass spectral similarity metric that can predict chemical similarity between mass spectra of not only the same, but also different ionization modes."
- [abstract] Embedding evaluator flags unreliable spectra: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [discussion] Unbalanced ionization mode sampling caveat: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] Adduct excluded from default model: "not all preprocessing tools generate reliable adduct information. Therefore, we decided to not include this in the default model"
- [readme] Training requires >100k spectra: [evidence span withheld — anchor status: external_doc]
- [readme] Pair sampling must be checked per dataset: [evidence span withheld — anchor status: external_doc]
- [readme] Concrete training invocation with metadata encoders: [evidence span withheld — anchor status: external_doc]
- [intro] PyTorch implementation: "the entire model was implemented using Pytorch25"
