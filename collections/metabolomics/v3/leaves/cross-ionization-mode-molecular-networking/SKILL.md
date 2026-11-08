---
name: cross-ionization-mode-molecular-networking
description: Use when you have MS/MS spectra acquired in both positive and negative
  ionization modes (e.g. mgf/msp from LC-MS metabolomics) and want a single molecular
  network spanning both modes rather than separate per-mode networks.
when_to_use_negative:
- Your spectra are all from a single ionization mode only — plain within-mode spectral-similarity
  networking suffices and cross-mode training is unnecessary overhead
- Input spectra are low quality with limited fragmentation, chimeric, or strongly
  dissimilar to the training libraries — the paper predicts poor MS2DeepScore scores
  for such spectra, so network edges would be unreliable
- You only need library-based annotation (e.g. MS2Query-style search) rather than
  network-level exploration of chemical space
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore 2.0
  role: Predicts cross-ionization-mode chemical similarity between tandem mass spectra
    to serve as network edge weights
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Provides the default spectral cleaning filters and the Pipeline used to compute
    the MS2DeepScore similarity matrix
- name: mzmine
  role: Offers local, no-code deployment of MS2DeepScore molecular networking for
    scientists without programming expertise
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:6096d3d9a1200ea86c0258c51fbb5b7a24c324e23cc5d3569ca185cd92bd2223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - matchms
  - mzmine
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-ionization-mode-molecular-networking

## Summary

Builds molecular networks that integrate tandem mass spectra from both positive and negative ionization modes into one unified graph, using MS2DeepScore 2.0-predicted chemical similarities instead of mode-restricted spectral cosine scores.

## When to use

Use when you have MS/MS spectra acquired in both positive and negative ionization modes (e.g. mgf/msp from LC-MS metabolomics) and want a single molecular network spanning both modes rather than separate per-mode networks.

## When NOT to use

- Your spectra are all from a single ionization mode only — plain within-mode spectral-similarity networking suffices and cross-mode training is unnecessary overhead
- Input spectra are low quality with limited fragmentation, chimeric, or strongly dissimilar to the training libraries — the paper predicts poor MS2DeepScore scores for such spectra, so network edges would be unreliable
- You only need library-based annotation (e.g. MS2Query-style search) rather than network-level exploration of chemical space

## Inputs

- Peak-processed MS/MS spectra in both positive and negative ionization modes (mgf, msp, mzML, mzXML, JSON, or USI formats accepted by matchms)
- Pre-trained MS2DeepScore 2.0 model weights (ms2deepscore_model.pt)
- Optional: quality/uncertainty scores from the embedding evaluator for each spectrum

## Outputs

- Pairwise chemical-similarity matrix (numpy array) covering same-mode and cross-mode spectrum pairs
- Unified molecular network graph (e.g. graphml for Cytoscape) with cross-ionization-mode clusters
- Optionally: UMAP visualization of MS2DeepScore embeddings as interactive chemical-space plot

## How to apply

First clean spectra with the matchms default filtering pipeline, then run the pre-trained MS2DeepScore 2.0 model to predict pairwise chemical similarities between all spectra, including cross-ionization-mode pairs — the model was trained on spectra from both modes so it predicts similarities across polarity boundaries. Optionally filter unreliable spectra before networking: the embedding evaluator flags spectra with low information content or spectra dissimilar to the training data, and the paper notes overall accuracy improves when high-uncertainty test spectra are removed. From the full similarity matrix, keep top-scoring links (the model's default pipeline uses top-20 mutual links with a similarity threshold around 0.85, as in the urine case study) and export the resulting graph (graphml) for visualization in Cytoscape. Because edges are predicted chemical similarities rather than fragmentation cosine matches, clusters can bridge positive/negative mode spectra of the same metabolite family, as demonstrated for the caffeine-metabolism cluster in the human urine case study.

## Related tools

- **MS2DeepScore 2.0** (Predicts cross-ionization-mode chemical similarity between tandem mass spectra to serve as network edge weights) — https://github.com/matchms/ms2deepscore
- **matchms** (Provides the default spectral cleaning filters and the Pipeline used to compute the MS2DeepScore similarity matrix)
- **mzmine** (Offers local, no-code deployment of MS2DeepScore molecular networking for scientists without programming expertise)

## Examples

```
from ms2deepscore import MS2DeepScore; from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; model = load_model('ms2deepscore_model.pt'); pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {'model': model}]])); report = pipeline.run('combined_spectra.mgf'); similarity_matrix = pipeline.scores.to_array()
```

## Evaluation signals

- The exported similarity matrix should be square with dimensions equal to the number of cleaned spectra, containing scores for both same-mode and cross-mode pairs
- Cross-ionization-mode edges should exist in the final graph — the urine case study confirms clusters containing spectra from both modes (e.g. the caffeine cluster)
- Spectra flagged with high predicted uncertainty (embedding evaluator) should be removed or down-weighted; the paper shows this raises overall accuracy
- Known biochemically related metabolite groups (e.g. caffeine-metabolism pathways) should appear as connected cross-mode clusters, verifiable by expert annotation

## Limitations

- MS2DeepScore predictions are unreliable for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to the training data
- The default model does not use adduct information because not all preprocessing tools generate reliable adduct metadata; incorrect adduct input could reduce performance
- There is a risk the model is not well-suited for very specific tasks or chemical classes outside its training distribution

## Evidence

- [abstract] MS2DeepScore 2.0 works between both different and same ionization modes: "which works both between different and the same ionization modes"
- [discussion] Similarity-based graphs can now be generated independent of the ionization mode: "Similarity- based graphs can now be generated independent of the ionization mode, rendering cross-ionization mode molecular networking feasible"
- [intro] Unifying positive and negative spectra via predicted similarities: "By using these predicted similarities by MS2DeepScore, both positive and negative ionization mode spectra can be visualized in one unified molecular network"
- [abstract] Quality filtering of unreliable spectra: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [other] Concrete network construction parameters from the urine case study: [evidence span withheld — anchor status: not_found]
- [intro] Model input includes metadata: "using ionization mode and precursor m/z as input improves model performance"
