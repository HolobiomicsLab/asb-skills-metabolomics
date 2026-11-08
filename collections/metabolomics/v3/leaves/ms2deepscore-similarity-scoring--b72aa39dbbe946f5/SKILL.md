---
name: ms2deepscore-similarity-scoring
description: Use when tandem MS/MS spectra must be compared across ionization modes
  (e.g., a Rutin positive-mode vs negative-mode pair), or when cosine (0.0) and modified
  cosine (0.3) scores underreport true chemical similarity of structurally identical
  compounds.
when_to_use_negative:
- Spectra are low quality, have limited fragmentation, are chimeric (multiple precursor
  ions), or differ substantially from the training data — poor predictions are expected
- The target compound classes are very specific or far outside the training data —
  there is a stated risk the model is not well-suited
- Only negative-vs-negative pairs in edge-case chemical space — pair sampling did
  not enforce balanced ionization-mode pairs, so negative vs negative performance
  may be limited
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network that predicts chemical (Tanimoto) similarity between
    spectrum pairs, including across ionization modes
  repo: github.com/matchms/MS2DeepScore
- name: matchms
  role: Provides the Pipeline, DEFAULT_FILTERS spectrum cleaning, and score-computation
    wrapper through which MS2DeepScore is run
  repo: github.com/matchms/matchms
- name: UMAP
  role: Dimensionality reduction for visualizing the MS2DeepScore embeddings in chemical
    space
- name: mzmine
  role: Local deployment of MS2DeepScore molecular networking for users without programming
    expertise
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:d705337d2f5e733285af1aab9d74e237fee57d761c55a2ffe2df3db426df59f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore
  - MS2DeepScore 2.0
  - UMAP
  - matchms
  - mzmine
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8518518518518519
n_claims: 27
---

# ms2deepscore-similarity-scoring

## Summary

Score chemical similarity between pairs of tandem mass spectra using MS2DeepScore 2.0, a Siamese neural network trained to predict Tanimoto-based structural similarity, including positive-vs-negative ionization mode pairs that classical cosine scores miss.

## When to use

Use when tandem MS/MS spectra must be compared across ionization modes (e.g., a Rutin positive-mode vs negative-mode pair), or when cosine (0.0) and modified cosine (0.3) scores underreport true chemical similarity of structurally identical compounds.

## When NOT to use

- Spectra are low quality, have limited fragmentation, are chimeric (multiple precursor ions), or differ substantially from the training data — poor predictions are expected
- The target compound classes are very specific or far outside the training data — there is a stated risk the model is not well-suited
- Only negative-vs-negative pairs in edge-case chemical space — pair sampling did not enforce balanced ionization-mode pairs, so negative vs negative performance may be limited

## Inputs

- Tandem mass spectra (MS/MS) in mgf, msp, mzml, mzxml, json or usi format, e.g. tests/resources/pesticides_processed.mgf
- Pretrained dual-mode model file ms2deepscore_model.pt (Zenodo record 17826815)
- Spectrum pairs from different ionization modes (e.g., Rutin positive and negative mode spectra from the Rumex sanguineus case study)

## Outputs

- Square numpy similarity matrix of MS2DeepScore predictions between all input spectra
- Per-spectrum embedding vectors usable for UMAP chemical-space visualization
- Embedding-evaluator quality/uncertainty estimates per input spectrum

## How to apply

Download the pretrained ms2deepscore_model.pt (trained on >500,000 MS/MS combined spectra from GNPS, MoNA, MassBank and MSnLib) and load it with ms2deepscore.models.load_model. Clean input spectra with the matchms DEFAULT_FILTERS pipeline, then run a matchms Pipeline with score_computations=[[MS2DeepScore, {'model': model}]] over the query file (mgf, msp, mzml, mzxml, json or usi formats are supported). The pipeline returns a square numpy similarity matrix of MS2DeepScore predictions between all spectra; intermediate embeddings can be extracted with MS2DeepScore(model).get_embedding_array() for UMAP visualization. For cross-mode pairs of structurally identical compounds (e.g., Rutin), the predicted similarity should be substantially higher than the cosine baseline and clearly higher than modified cosine. Check the embedding evaluator output and filter spectra with high predicted uncertainty before trusting predictions.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network that predicts chemical (Tanimoto) similarity between spectrum pairs, including across ionization modes) — github.com/matchms/MS2DeepScore
- **matchms** (Provides the Pipeline, DEFAULT_FILTERS spectrum cleaning, and score-computation wrapper through which MS2DeepScore is run) — github.com/matchms/matchms
- **UMAP** (Dimensionality reduction for visualizing the MS2DeepScore embeddings in chemical space)
- **mzmine** (Local deployment of MS2DeepScore molecular networking for users without programming expertise)

## Examples

```
model = load_model("ms2deepscore_model.pt"); pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); pipeline.run("pesticides.mgf")
```

## Evaluation signals

- For the structurally identical Rutin positive/negative mode pair, MS2DeepScore 2.0 should predict ~0.7 while cosine gives 0.0 and modified cosine 0.3
- The output is a square numpy similarity matrix with one pairwise prediction per spectrum pair
- Spectra flagged by the embedding evaluator with high predicted MSE/uncertainty are excluded before predictions are trusted
- Predicted similarities approximate expected Tanimoto structural similarities of the underlying molecules

## Limitations

- Poor predictions expected for low-quality spectra with limited fragmentation, chimeric spectra from multiple precursor ions, or spectra dissimilar to the training data
- The new pair sampling algorithm did not enforce balanced sampling of ionization-mode pairs, possibly limiting negative-vs-negative mode performance
- Fragment signals are binned, which might result in the loss of useful information; adduct metadata is excluded from the default model because not all preprocessing tools generate reliable adduct information

## Evidence

- [readme] Siamese neural network trained on Tanimoto targets: "provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
- [readme] Cross-mode capability: [evidence span withheld — anchor status: external_doc]
- [abstract] MS2DeepScore 2.0 definition: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [intro] Embedding quality estimation: "we introduce a method that can estimate the mass spectral embedding quality for each input spectrum"
- [discussion] Uncertainty filtering improves accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
