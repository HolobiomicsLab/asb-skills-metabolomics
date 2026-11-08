---
name: cross-ionization-mode-similarity-prediction
description: Use when positive and negative ionization mode MS/MS spectra of structurally
  identical or related compounds must be compared, and classical spectrum similarity
  scores (cosine, modified cosine) miss cross-mode similarity.
when_to_use_negative:
- When all spectra are within a single ionization mode and classical spectral similarity
  scores suffice — the cross-mode capability is the reason to use this model
- 'When the analysis depends on adduct information: the default model excludes adduct
  input because not all preprocessing tools generate reliable adduct information,
  which could reduce performance'
- When training a custom model on a small dataset (<100,000 spectra) without first
  merging with larger public libraries — pair sampling can be suboptimal if not re-checked
  and re-optimized
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore 2.0
  role: Pre-trained Siamese neural network that predicts Tanimoto chemical similarity
    between spectra pairs, including across positive and negative ionization modes
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Provides the spectral cleaning pipeline (DEFAULT_FILTERS) and Pipeline/create_workflow
    machinery that wraps MS2DeepScore score computation over input spectra
  repo: https://github.com/matchms/matchms
- name: PyTorch
  role: Framework in which the entire MS2DeepScore 2.0 model is implemented; required
    backend for loading and running the model
- name: UMAP
  role: Dimensionality reduction applied to MS2DeepScore embeddings to visualize spectra
    in chemical space via an interactive plot
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:d437182af203ef42ddb714f87472dfc067f203807061ebcc8c99fb0eff745510
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - UMAP
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-ionization-mode-similarity-prediction

## Summary

MS2DeepScore 2.0 is a Siamese neural network, implemented in PyTorch and trained on >500,000 MS/MS spectra from GNPS, MoNA, MassBank and MSnLib, that predicts Tanimoto chemical similarity between tandem mass spectra acquired in both the same and different ionization modes. It enables unified positive/negative-mode molecular networking where classical scores (cosine, modified cosine) fail across polarities.

## When to use

Use when positive and negative ionization mode MS/MS spectra of structurally identical or related compounds must be compared, and classical spectrum similarity scores (cosine, modified cosine) miss cross-mode similarity.

## When NOT to use

- When all spectra are within a single ionization mode and classical spectral similarity scores suffice — the cross-mode capability is the reason to use this model
- When the analysis depends on adduct information: the default model excludes adduct input because not all preprocessing tools generate reliable adduct information, which could reduce performance
- When training a custom model on a small dataset (<100,000 spectra) without first merging with larger public libraries — pair sampling can be suboptimal if not re-checked and re-optimized

## Inputs

- Tandem mass spectra (MS/MS) in positive and negative ionization modes, in formats supported by matchms (msp, mzml, mgf, mzxml, json, usi)
- Pre-trained MS2DeepScore 2.0 model file ms2deepscore_model.pt (Zenodo record 17826815)
- Optional: reference Tanimoto similarities for a test set when evaluating accuracy or the embedding evaluator

## Outputs

- N x N similarity matrix (numpy array) of MS2DeepScore-predicted Tanimoto scores across same- and cross-ionization-mode spectrum pairs
- MS2DeepScore embedding arrays for UMAP visualization of chemical space
- Per-spectrum predicted MSE from the embedding evaluator used to flag/filter unreliable embeddings

## How to apply

Load the pre-trained dual-mode model (ms2deepscore_model.pt from Zenodo record 17826815) with ms2deepscore.models.load_model and clean spectra using matchms DEFAULT_FILTERS. Wrap the model in MS2DeepScore and compute all-pairs predicted Tanimoto similarities, or extract per-spectrum embeddings via get_embedding_array for UMAP visualization of chemical space. When scoring a cross-mode pair of a structurally identical compound, benchmark against classical baselines: for the Rutin positive-vs-negative pair in the Rumex sanguineus case study, MS2DeepScore 2.0 predicted 0.7 while modified cosine gave 0.3 and cosine gave 0.0. Optionally run the InceptionTime embedding evaluator on each embedding to get a per-spectrum predicted MSE, rank spectra descending by predicted MSE, and remove the most uncertain fraction — accuracy (RMSE vs true Tanimoto) on remaining spectra increases (Figure 4c). For custom data, retrain via train_ms2deepscore_wrapper with SettingsMS2Deepscore, passing ionmode (CategoricalToBinary) and standardized precursor_mz (mean 0, standard deviation 1000) as additional metadata, since adding ionization mode and precursor m/z improves model performance.

## Related tools

- **MS2DeepScore 2.0** (Pre-trained Siamese neural network that predicts Tanimoto chemical similarity between spectra pairs, including across positive and negative ionization modes) — https://github.com/matchms/ms2deepscore
- **matchms** (Provides the spectral cleaning pipeline (DEFAULT_FILTERS) and Pipeline/create_workflow machinery that wraps MS2DeepScore score computation over input spectra) — https://github.com/matchms/matchms
- **PyTorch** (Framework in which the entire MS2DeepScore 2.0 model is implemented; required backend for loading and running the model)
- **UMAP** (Dimensionality reduction applied to MS2DeepScore embeddings to visualize spectra in chemical space via an interactive plot)

## Examples

```
model = load_model("ms2deepscore_model.pt"); pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); pipeline.run(spectrum_file_name)
```

## Evaluation signals

- For the structurally identical Rutin compound measured in positive vs negative mode, predicted similarity should be substantially higher than the cosine baseline of 0.0 and clearly higher than the modified cosine score of 0.3 (expected ≈ 0.7)
- The resulting similarity matrix should be an N x N numpy array with predicted Tanimoto values on the [0, 1] scale
- Iteratively removing the highest-predicted-MSE spectra (embedding evaluator) should monotonically improve prediction accuracy (lower RMSE between predicted and true Tanimoto similarity) on the remaining spectra
- Spectra must pass the matchms DEFAULT_FILTERS cleaning pipeline before embedding; spectra with low information content or dissimilar to training data should be flagged

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to the training data
- The new pair sampling algorithm did not enforce balanced sampling of different ionization mode pairs, which might limit negative vs negative mode prediction performance
- Fragment signals are initially binned, which might result in loss of useful information; MS1 data is not included because public mass spectral libraries lack annotated raw MS1 spectra
- There remains a risk that the model is not well-suited for very specific tasks or chemical classes

## Evidence

- [abstract] MS2DeepScore 2.0 predicts cross-mode Tanimoto similarity: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [intro] Model works between same and different ionization modes: "Here, we developed a mass spectral similarity metric that can predict chemical similarity between mass spectra of not only the same, but also different ionization modes."
- [readme] Siamese network predicts Tanimoto from spectrum pairs: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
- [discussion] Uncertainty filtering raises accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [discussion] Poor predictions on low-quality spectra: "Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions"
