---
name: tanimoto-similarity-prediction-evaluation
description: Use when a mass spectra library with reference structures is available
  and you must quantify chemical similarity prediction quality, e.g., RMSE between
  predicted and reference Tanimoto scores on held-out spectrum pairs.
when_to_use_negative:
- Training a new model on a small custom dataset (<100,000 spectra) — the article
  recommends merging with larger public libraries instead, since pair sampling is
  likely suboptimal
- Spectra with low fragmentation quality, chimeric spectra, or spectra far from the
  training data — filter them first using the embedding evaluator's predicted MSE
- Very specific tasks or narrow chemical classes, where the general model risks being
  poorly suited
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3820
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network that predicts Tanimoto molecular structural similarity
    from pairs of mass spectrometry spectra; the prediction target evaluated by this
    skill
  repo: github.com/matchms/ms2deepscore
- name: matchms
  role: Provides the DEFAULT_FILTERS cleaning pipeline and Pipeline workflow wrapper
    used to preprocess spectra before similarity computation
- name: PyTorch
  role: Backend in which the Siamese model is implemented and trained with balanced
    pair sampling over Tanimoto bins
- name: ms2deepscore (SettingsMS2Deepscore)
  role: Defines training/evaluation settings including additional metadata inputs
    (ionmode, precursor_mz) and ionisation_mode='both'
  repo: pypi.org/project/ms2deepscore
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:a781c6549e5d598f924b7dcf924d0074fa7b177e43ed696e7220ea68671396d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - SettingsMS2Deepscore
  - matchms
  - ms2deepscore (SettingsMS2Deepscore)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3820
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tanimoto-similarity-prediction-evaluation

## Summary

Evaluates how well a Siamese neural network (MS2DeepScore 2.0) predicts Tanimoto chemical structural similarity between tandem mass spectra, including across positive and negative ionization modes, by comparing predicted similarities to reference Tanimoto scores.

## When to use

Use when a mass spectra library with reference structures is available and you must quantify chemical similarity prediction quality, e.g., RMSE between predicted and reference Tanimoto scores on held-out spectrum pairs.

## When NOT to use

- Training a new model on a small custom dataset (<100,000 spectra) — the article recommends merging with larger public libraries instead, since pair sampling is likely suboptimal
- Spectra with low fragmentation quality, chimeric spectra, or spectra far from the training data — filter them first using the embedding evaluator's predicted MSE
- Very specific tasks or narrow chemical classes, where the general model risks being poorly suited

## Inputs

- Annotated tandem mass spectra (mgf, msp, mzml, mzxml, json, usi) with per-spectrum metadata (precursor m/z, ionmode, adduct type, instrument type)
- Reference chemical structures for computing Tanimoto similarity labels
- Pre-trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt from zenodo.org/records/17826815)

## Outputs

- Similarity matrix (numpy array) of predicted Tanimoto scores between all spectra
- Per-variant RMSE comparison table (e.g., metadata_input_ablation.csv)
- Removal-fraction vs accuracy table for uncertainty filtering (e.g., figure4c_mse_filtering.csv)

## How to apply

Load the pre-trained MS2DeepScore 2.0 model (ms2deepscore_model.pt from Zenodo, trained on >500,000 combined MS/MS spectra from GNPS, Mona, MassBank and MSnLib) and clean input tandem mass spectra with the matchms DEFAULT_FILTERS pipeline before prediction. Compute embeddings and predicted Tanimoto similarities for all spectrum pairs spanning both positive and negative ionization modes. Train or fine-tune variants via SettingsMS2Deepscore using the pair sampling algorithm that balances pairs over Tanimoto similarity bins, optionally adding metadata (precursor m/z, ionmode via CategoricalToBinary, adduct type) as input. Evaluate each variant by computing RMSE between predicted and reference Tanimoto similarities on held-out test pairs, optionally removing spectra flagged by the InceptionTime embedding evaluator with high predicted MSE to verify accuracy gains. Compile per-variant RMSE comparisons (e.g., metadata_input_ablation.csv, figure4c_mse_filtering.csv) and confirm expected trends: lower RMSE for precursor m/z + ionmode/adduct variants, improving accuracy as high-MSE spectra are removed.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network that predicts Tanimoto molecular structural similarity from pairs of mass spectrometry spectra; the prediction target evaluated by this skill) — github.com/matchms/ms2deepscore
- **matchms** (Provides the DEFAULT_FILTERS cleaning pipeline and Pipeline workflow wrapper used to preprocess spectra before similarity computation)
- **PyTorch** (Backend in which the Siamese model is implemented and trained with balanced pair sampling over Tanimoto bins)
- **ms2deepscore (SettingsMS2Deepscore)** (Defines training/evaluation settings including additional metadata inputs (ionmode, precursor_mz) and ionisation_mode='both') — pypi.org/project/ms2deepscore

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; from ms2deepscore import MS2DeepScore; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); pipeline.run(spectrum_file_name)
```

## Evaluation signals

- RMSE between predicted and reference Tanimoto similarities on held-out test pairs is lower for variants with precursor m/z + ionization mode and adduct type than the baseline, while the one-hot instrument-type variant does not improve over baseline
- Accuracy (RMSE) on remaining spectra increases monotonically as the highest predicted-MSE spectra are removed, reproducing the Figure 4c trend
- The output similarity matrix is a square numpy array containing predictions between all input spectra, with values in the 0–1 Tanimoto range
- The model produces valid predictions for cross-ionization-mode pairs (positive vs negative), not only same-mode pairs
- Embedding evaluator flags spectra with low information content or dissimilarity to training data as unreliable before predictions are used

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to training data
- Fragment signals are initially binned, which might result in the loss of useful information
- The new pair sampling algorithm did not enforce balanced sampling of different ionization mode pairs, potentially limiting negative vs negative prediction performance; adduct input is excluded from the default model because preprocessing tools do not always generate reliable adduct information

## Evidence

- [readme] Siamese network predicts Tanimoto: "ms2deepscore provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
- [discussion] Accuracy improves by removing high-uncertainty spectra: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [intro] Metadata inputs improve performance: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [abstract] Cross-mode prediction capability: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [abstract] Embedding evaluator flags unreliable spectra: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
