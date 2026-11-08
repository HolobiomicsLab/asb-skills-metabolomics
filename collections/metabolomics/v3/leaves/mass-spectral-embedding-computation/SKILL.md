---
name: mass-spectral-embedding-computation
description: Use when you have preprocessed MS/MS spectra (mgf, msp, mzml, mzxml,
  json, usi) in positive and/or negative ionization mode and need a numeric representation
  for cross-mode similarity scoring or chemical-space visualization.
when_to_use_negative:
- Training a new model from scratch on a small custom dataset without merging it with
  larger public libraries — pair sampling can be suboptimal if not checked for smaller
  training sets
- Spectra are low-quality with limited fragmentation, chimeric, or substantially dissimilar
  to training data — predictions are unreliable; use the embedding evaluator to flag
  these before trusting embeddings
- The DreaMS model is required instead (it has only been trained on positive ionization
  spectra, so is unsuitable for cross-mode training)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2DeepScore
  role: Siamese neural network library that computes embeddings and predicted Tanimoto
    similarities for mass spectra
  repo: github.com/matchms/ms2deepscore
- name: MS2DeepScore 2.0
  role: Cross-ionization-mode model version used to compute embeddings for both positive
    and negative mode spectra
  repo: github.com/matchms/ms2deepscore
- name: matchms
  role: Provides the DEFAULT_FILTERS preprocessing pipeline and Pipeline framework
    used to feed cleaned spectra into the embedding model
- name: UMAP
  role: Dimensionality reduction of 500-dim embeddings to 2D for chemical-space visualization
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:5ed4cceafa0ab4b5205a14d556b42b614ac01634558b38077b848aeef5d38b17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore
  - MS2DeepScore 2.0
  - UMAP
  - matchms
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectral-embedding-computation

## Summary

Computes 500-dimensional MS2DeepScore 2.0 embeddings for preprocessed tandem mass spectra using a Siamese neural network that predicts Tanimoto-based chemical similarity, serving as both an intermediate for score computation and a basis for UMAP visualization of chemical space.

## When to use

Use when you have preprocessed MS/MS spectra (mgf, msp, mzml, mzxml, json, usi) in positive and/or negative ionization mode and need a numeric representation for cross-mode similarity scoring or chemical-space visualization.

## When NOT to use

- Training a new model from scratch on a small custom dataset without merging it with larger public libraries — pair sampling can be suboptimal if not checked for smaller training sets
- Spectra are low-quality with limited fragmentation, chimeric, or substantially dissimilar to training data — predictions are unreliable; use the embedding evaluator to flag these before trusting embeddings
- The DreaMS model is required instead (it has only been trained on positive ionization spectra, so is unsuitable for cross-mode training)

## Inputs

- Preprocessed tandem mass spectra in positive and/or negative ionization mode (e.g. .mgf, .msp, .mzML, .mzXML, .json, .usi)
- Trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt, downloadable from Zenodo record 17826815)
- matchms filtering pipeline applied to raw spectra before embedding

## Outputs

- 500-dimensional embedding vector for each input spectrum (numpy array from get_embedding_array)
- Similarity matrix of predicted chemical (Tanimoto) scores between all spectrum pairs (pipeline.scores.to_array())
- Optionally a 2D UMAP projection and interactive HTML plot of embeddings colored by ionization mode

## How to apply

Load the pretrained MS2DeepScore 2.0 model (ms2deepscore_model.pt from Zenodo record 17826815), which was trained on >500,000 MS/MS spectra from GNPS, Mona, MassBank and MSnLib and works across both ionization modes. First clean spectra with the matchms DEFAULT_FILTERS pipeline, then wrap the model in the MS2DeepScore class and run the matchms Pipeline with score_computations=[[MS2DeepScore, {"model": model}]] to obtain a similarity matrix of predicted Tanimoto scores for all spectrum pairs. To obtain embeddings themselves, call ms2ds_model.get_embedding_array(cleaned_spectra) on the cleaned spectra; each spectrum is encoded into a 500-dimensional vector. For visualization, reduce these embeddings to 2D with UMAP (e.g. n_neighbors=50 in the Rumex sanguineus case study) and optionally render an interactive HTML plot colored by ionization mode to confirm positive and negative mode spectra intermix.

## Related tools

- **MS2DeepScore** (Siamese neural network library that computes embeddings and predicted Tanimoto similarities for mass spectra) — github.com/matchms/ms2deepscore
- **MS2DeepScore 2.0** (Cross-ionization-mode model version used to compute embeddings for both positive and negative mode spectra) — github.com/matchms/ms2deepscore
- **matchms** (Provides the DEFAULT_FILTERS preprocessing pipeline and Pipeline framework used to feed cleaned spectra into the embedding model)
- **UMAP** (Dimensionality reduction of 500-dim embeddings to 2D for chemical-space visualization)

## Examples

```
from ms2deepscore import MS2DeepScore; model = load_model('ms2deepscore_model.pt'); embeddings = MS2DeepScore(model).get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- Similarity matrix is a square numpy array of shape n_spectra x n_spectra with values in [0,1] representing predicted Tanimoto scores
- For structurally identical molecules measured in different ionization modes (e.g. Rutin positive vs. negative), predicted similarity should be substantially higher than cosine (0.0) and modified cosine (0.3) baselines — MS2DeepScore gave 0.7
- UMAP of embeddings with n_neighbors=50 should show positive and negative ionization mode spectra intermixed, confirming cross-mode chemical similarity is captured
- Spectra flagged by the embedding evaluator (low information content or dissimilar to training data) should be filtered out before downstream use to minimize false predictions

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to the training data
- The model's predicted uncertainty should be monitored; removing spectra with high predicted uncertainty raises overall accuracy
- New pair sampling algorithm did not enforce balanced sampling of the different ionization mode pairs, which may limit negative vs. negative mode prediction performance
- Adduct information is excluded from the default model because not all preprocessing tools generate reliable adduct data
- There remains a risk that the model is not well-suited for very specific tasks or chemical classes

## Evidence

- [readme] Compute spectral similarities: "The library provides intuitive classes to prepare data, train a Siamese model, and compute similarities between pairs of spectra."
- [readme] Embedding intermediate product: [evidence span withheld — anchor status: external_doc]
- [abstract] Cross-ionization-mode prediction: "which works both between different and the same ionization modes"
- [other] Rutin cross-mode score: [evidence span withheld — anchor status: not_found]
- [readme] Embedding evaluator: [evidence span withheld — anchor status: external_doc]
