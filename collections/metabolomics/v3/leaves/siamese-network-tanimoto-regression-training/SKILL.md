---
name: siamese-network-tanimoto-regression-training
description: Use when you need to train (or fine-tune) a Siamese network that predicts
  chemical (Tanimoto) similarity from tandem MS spectrum pairs, especially when spectra
  span both positive and negative ionization modes.
when_to_use_negative:
- Your task requires cross-ionization-mode similarity from a model trained only on
  positive-mode spectra (e.g. the DreaMS model was trained only on positive ionization
  and is unsuitable for cross-mode training)
- You have a small dataset and want reliable training without first merging it with
  larger public libraries or fine-tuning the pre-trained model — pair sampling can
  be suboptimal on small training sets
- You only need to score existing spectra — use the pre-trained model for prediction
  instead of retraining
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: ms2deepscore
  role: Library providing the Siamese network architecture, balanced pair-sampling
    settings (SettingsMS2Deepscore) and training wrapper used to train the Tanimoto-regression
    model
  repo: github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the Siamese neural network model is implemented and trained
- name: matchms
  role: Cleaning/filtering pipeline applied to spectra before training; also drives
    score computation with the trained MS2DeepScore model
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:3e53228c13673b2570bcc554bd16c603b0ceba147d489e1b44506680f6b9a620
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# siamese-network-tanimoto-regression-training

## Summary

Train a Siamese neural network (MS2DeepScore 2.0) in PyTorch to regress Tanimoto structural similarity from pairs of tandem MS spectra, using ionization mode and precursor m/z as additional model inputs so predictions work both within and across positive/negative ionization modes.

## When to use

Use when you need to train (or fine-tune) a Siamese network that predicts chemical (Tanimoto) similarity from tandem MS spectrum pairs, especially when spectra span both positive and negative ionization modes.

## When NOT to use

- Your task requires cross-ionization-mode similarity from a model trained only on positive-mode spectra (e.g. the DreaMS model was trained only on positive ionization and is unsuitable for cross-mode training)
- You have a small dataset and want reliable training without first merging it with larger public libraries or fine-tuning the pre-trained model — pair sampling can be suboptimal on small training sets
- You only need to score existing spectra — use the pre-trained model for prediction instead of retraining

## Inputs

- Tandem MS spectra from GNPS, MoNA, MassBank EU and MSnLib (mgf, msp, mzML, mzXML, json, usi; e.g. combined_libraries.mgf)
- Known chemical structures per spectrum for computing ground-truth Tanimoto similarity labels
- Spectrum metadata: precursor m/z and ionization mode (positive/negative)

## Outputs

- Trained Siamese network model file (e.g. ms2deepscore_model.pt, downloadable from Zenodo record 17826815)
- Embedding evaluator model for estimating per-spectrum prediction reliability
- Spectrum embeddings usable for UMAP visualization and similarity matrices of predicted Tanimoto scores

## How to apply

Retrieve tandem MS spectra from public libraries (GNPS, MoNA, MassBank EU, MSnLib), clean them with the matchms filtering pipeline, and derive ground-truth Tanimoto labels from known structures. Use the balanced pair-sampling algorithm (SettingsMS2Deepscore) that stratifies spectrum pairs over Tanimoto similarity bins to reduce training bias, then train the Siamese network via train_ms2deepscore_wrapper with ionisation_mode='both' and additional_metadata supplying ionmode (CategoricalToBinary) and standardized precursor m/z (StandardScaler, mean 0, sd 1000). Optionally also train the embedding evaluator (SettingsEmbeddingEvaluator) so unreliable embeddings can later be flagged. For smaller custom datasets, merge in-house spectra with the public libraries before training from scratch, or fine-tune the pre-trained model instead; check pair-sampling quality and re-optimize for new datasets since it can be suboptimal for small training sets.

## Related tools

- **ms2deepscore** (Library providing the Siamese network architecture, balanced pair-sampling settings (SettingsMS2Deepscore) and training wrapper used to train the Tanimoto-regression model) — github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the Siamese neural network model is implemented and trained)
- **matchms** (Cleaning/filtering pipeline applied to spectra before training; also drives score computation with the trained MS2DeepScore model) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; train_ms2deepscore_wrapper(SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'})], ionisation_mode='both'), SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Held-out test set shows strong correlation between predicted and true Tanimoto similarity for cross-ionization-mode pairs (Figure 2b reproduction), with Pearson r and RMSE saved to correlation_metrics.csv
- Training pair counts are balanced across Tanimoto similarity bins (check pair-sampling tutorial output before training)
- additional_metadata settings correctly encode ionmode as binary (positive=1, negative=0) and precursor m/z standardized with mean 0, standard deviation 1000
- Similarity matrix output from pipeline.run() is a complete numpy array over all spectrum pairs and computable for positive-positive, negative-negative, and cross-mode pairs
- Embedding evaluator, if trained, flags low-information or out-of-distribution spectra rather than producing silent false predictions

## Limitations

- The new pair sampling algorithm did not enforce balanced sampling of the different ionization mode pairs; more balanced ionization-mode sampling might improve negative-vs-negative mode performance, and Tanimoto score distribution differences can leave unequal numbers of pairs per bin per ionization mode
- Fragment signals are binned, which might result in the loss of useful information; adduct information was excluded from the default model because preprocessing tools do not always produce reliable adduct annotations
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra from multiple precursor ions, or spectra dissimilar to the training data; there remains a risk the model is not well-suited for very specific tasks or chemical classes
- Training your own model requires a substantial dataset (README suggests > 100,000 diverse spectra) and machine-learning familiarity

## Evidence

- [abstract] we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [intro] New pair sampling algorithm balances pairs over Tanimoto bins: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [intro] Metadata inputs improve performance: "using ionization mode and precursor m/z as input improves model performance"
- [intro] PyTorch implementation: "the entire model was implemented using Pytorch25"
- [readme] README: Siamese network predicts Tanimoto scores: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) 
from pairs of mass spectrometry spectra."
- [readme] README: training wrapper with ionmode and precursor m/z metadata: [evidence span withheld — anchor status: external_doc]
- [discussion] Merge small datasets before training from scratch: "For smaller custom datasets, we recommend merging them with larger available datasets, such as the here-used public libraries, before training a new model from scratch"
- [discussion] Ionization-mode pair sampling not balanced: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
