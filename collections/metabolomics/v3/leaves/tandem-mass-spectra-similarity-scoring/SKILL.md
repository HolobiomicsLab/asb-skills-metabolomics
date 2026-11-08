---
name: tandem-mass-spectra-similarity-scoring
description: Use when you have positive and/or negative ionization mode MS/MS peak
  spectra (mgf, msp, mzML, mzXML, json, usi) and need predicted chemical similarity
  scores, e.g., to build a unified cross-mode molecular network.
when_to_use_negative:
- The task is structure annotation from a single spectrum (SIRIUS, MS-Finder or MS2Query
  are the appropriate annotation tools, not a pairwise similarity metric)
- 'Only negative-vs-negative mode similarity at very low scores is critical: pair
  sampling over ionization modes was unbalanced, so negative/negative prediction performance
  may be limited'
- 'The spectra are of a very specific chemical class or task: the paper notes a risk
  that the general model is not well-suited to very specific tasks or chemical classes'
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network that predicts Tanimoto-based chemical similarity between
    spectra, including across different ionization modes
  repo: github.com/matchms/MS2DeepScore
- name: matchms
  role: Spectral cleaning pipeline (DEFAULT_FILTERS) and Pipeline/score-computation
    framework wrapping MS2DeepScore to produce the similarity matrix
  repo: https://github.com/matchms/matchms
- name: mzmine
  role: Local deployment combining MS2DeepScore-based molecular networking with feature
    detection for users without programming expertise
- name: UMAP
  role: Dimensionality reduction of MS2DeepScore embeddings for interactive chemical-space
    visualization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:8d1c5219c23f8f05ade880eaf2727ee501623896df9181ed4d8bebe372e549e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - UMAP
  - matchms
  - mzmine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tandem-mass-spectra-similarity-scoring

## Summary

Predicts pairwise chemical similarity (Tanimoto-score proxy) between tandem mass spectra — including spectra acquired in different ionization modes — using the MS2DeepScore 2.0 Siamese neural network, enabling cross-mode molecular networking.

## When to use

Use when you have positive and/or negative ionization mode MS/MS peak spectra (mgf, msp, mzML, mzXML, json, usi) and need predicted chemical similarity scores, e.g., to build a unified cross-mode molecular network.

## When NOT to use

- The task is structure annotation from a single spectrum (SIRIUS, MS-Finder or MS2Query are the appropriate annotation tools, not a pairwise similarity metric)
- Only negative-vs-negative mode similarity at very low scores is critical: pair sampling over ionization modes was unbalanced, so negative/negative prediction performance may be limited
- The spectra are of a very specific chemical class or task: the paper notes a risk that the general model is not well-suited to very specific tasks or chemical classes

## Inputs

- Tandem MS/MS peak spectra in positive and/or negative ionization mode (mgf, msp, mzml, mzxml, json, usi formats supported)
- Pre-trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt, downloadable from Zenodo record 17826815)

## Outputs

- Symmetric pairwise similarity matrix (numpy array) of MS2DeepScore predictions between all spectra
- Per-spectrum embedding vectors for dimensionality-reduction visualization (e.g., interactive UMAP)
- Molecular network graph spanning both ionization modes (exported as graphml for Cytoscape)
- Per-spectrum embedding-quality estimates flagging unreliable predictions

## How to apply

Clean the spectra with the matchms DEFAULT_FILTERS pipeline, then load the pre-trained MS2DeepScore 2.0 model (ms2deepscore_model.pt, trained on >500,000 GNPS/Mona/MassBank/MSnLib spectra with ionmode and precursor m/z as additional metadata input) and compute the full pairwise similarity matrix via a matchms Pipeline score computation. Optionally extract per-spectrum embeddings (get_embedding_array) for UMAP visualization of chemical space. For molecular networking, filter predicted similarities to each spectrum's top-20 links above a 0.85 cutoff, keep only mutual links (edges where each spectrum appears in the other's top-20), and export the unified positive+negative mode graph as graphml for Cytoscape. Before trusting predictions, use the embedding evaluator to flag spectra with low information content or dissimilarity to training data; spectra with high predicted uncertainty (predicted MSE) can be removed to raise overall accuracy. Validate by checking that cross-ionization-mode clusters are recovered, e.g., the caffeine-metabolism cluster spanning both modes confirmed by expert annotation.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network that predicts Tanimoto-based chemical similarity between spectra, including across different ionization modes) — github.com/matchms/MS2DeepScore
- **matchms** (Spectral cleaning pipeline (DEFAULT_FILTERS) and Pipeline/score-computation framework wrapping MS2DeepScore to produce the similarity matrix) — https://github.com/matchms/matchms
- **mzmine** (Local deployment combining MS2DeepScore-based molecular networking with feature detection for users without programming expertise)
- **UMAP** (Dimensionality reduction of MS2DeepScore embeddings for interactive chemical-space visualization)

## Examples

```
from ms2deepscore import MS2DeepScore; from ms2deepscore.models import load_model; from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; model = load_model("ms2deepscore_model.pt"); pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); pipeline.run("spectra.mgf"); S = pipeline.scores.to_array()
```

## Evaluation signals

- Similarity matrix is square and symmetric with all predicted scores in the expected 0–1 range of Tanimoto-score proxies
- Network edges survive the two-stage filter: only mutual top-20 links with predicted similarity above the 0.85 cutoff are retained
- The resulting graph integrates spectra from both ionization modes into cross-mode clusters (not two disconnected single-mode components)
- Known biochemically relevant clusters are recovered, e.g., the caffeine-related cluster matches known caffeine metabolism pathways under expert annotation
- Spectra flagged by the embedding evaluator (high predicted MSE / low information content) are excluded before downstream interpretation

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra from multiple precursor ions, or spectra dissimilar to the training data
- Fragment signals are binned before modeling, which might result in loss of useful information
- Adduct information is not included in the default model because not all preprocessing tools generate reliable adduct annotations, and incorrect adduct info could reduce performance
- The new pair sampling algorithm did not enforce balanced sampling of ionization-mode pairs, which might limit negative vs negative mode prediction performance

## Evidence

- [abstract] MS2DeepScore 2.0 predicts cross-mode chemical similarity: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [readme] Model handles both modes and cross-mode predictions: [evidence span withheld — anchor status: external_doc]
- [readme] Siamese network trained to predict Tanimoto scores from spectra pairs: "provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
- [discussion] Cross-mode molecular networking is enabled by the similarity graphs: "Similarity- based graphs can now be generated independent of the ionization mode, rendering cross-ionization mode molecular networking feasible"
- [intro] Embedding-quality filtering of unreliable predictions: "This allows users to filter out spectra for which the MS2DeepScore predictions are unreliable, e.g., due to low spectral quality or when spectra differ substantially from the training data."
- [intro] Additional metadata input improves model performance: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
