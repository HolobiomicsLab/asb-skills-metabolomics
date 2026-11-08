---
name: molecular-network-edge-filtering
description: Use when you have a full pairwise MS2DeepScore similarity matrix over
  spectra from both ionization modes and need to build a sparse, Cytoscape-ready molecular
  network of cross- and within-mode clusters.
when_to_use_negative:
- Input spectra have high predicted embedding uncertainty (predicted MSE) or low information
  content — filter them out first, as the paper warns predictions are unreliable for
  low-quality, chimeric, or training-dissimilar spectra
- The goal is model training, fine-tuning, or pair sampling optimization rather than
  constructing a network from already-computed similarities
- A different similarity measure (e.g. cosine) or a within-mode-only analysis is required
  and cross-ionization-mode integration is not the objective
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MS2DeepScore 2.0
  role: Predicts pairwise chemical (Tanimoto-based) similarities between spectra,
    including across ionization modes, providing the edge weights for the network
  repo: github.com/matchms/MS2DeepScore
- name: matchms Pipeline
  role: Applies default spectrum filters and runs MS2DeepScore score computations
    to produce the similarity matrix used for edge selection
  repo: github.com/matchms/ms2deepscore
- name: mzmine
  role: Alternative local deployment combining feature detection with MS2DeepScore-based
    molecular networking for users without programming expertise
  repo: github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:b7236ca120b8b8fad7ec5ff2be245ddfeac0156653444f147d70908d30164f80
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - matchms Pipeline
  - mzmine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-network-edge-filtering

## Summary

Constructs a unified molecular network from MS2DeepScore-predicted spectral similarities by keeping only top-20 links per spectrum above a 0.85 similarity cutoff and retaining mutually matched edges, so positive and negative ionization mode spectra can be integrated into one graph.

## When to use

Use when you have a full pairwise MS2DeepScore similarity matrix over spectra from both ionization modes and need to build a sparse, Cytoscape-ready molecular network of cross- and within-mode clusters.

## When NOT to use

- Input spectra have high predicted embedding uncertainty (predicted MSE) or low information content — filter them out first, as the paper warns predictions are unreliable for low-quality, chimeric, or training-dissimilar spectra
- The goal is model training, fine-tuning, or pair sampling optimization rather than constructing a network from already-computed similarities
- A different similarity measure (e.g. cosine) or a within-mode-only analysis is required and cross-ionization-mode integration is not the objective

## Inputs

- MGF/msp/mzML etc. peak spectra files containing positive and negative ionization mode spectra (e.g. urine case study spectra)
- Pre-trained MS2DeepScore 2.0 model file (e.g. ms2deepscore_model.pt from Zenodo)
- Full pairwise MS2DeepScore-predicted similarity matrix (numpy array from matchms Pipeline scores)

## Outputs

- Filtered edge list of mutual top-20 links with predicted similarity > 0.85
- Unified cross-ionization-mode molecular network graph exported in graphml format for Cytoscape

## How to apply

First compute the pairwise MS2DeepScore 2.0 similarity matrix across all spectra (positive and negative mode together) with the pre-trained model. For each spectrum, rank its predicted similarities and keep only the top-20 highest-scoring candidate links with a similarity above the 0.85 cutoff. Then keep only mutual links — edges where each spectrum also appears in the other spectrum's top-20 list — to define the final network edges; this reciprocity filter suppresses one-sided hub edges. Optionally, before scoring, drop spectra flagged by the embedding evaluator as unreliable (high predicted MSE or low information content), since the paper shows accuracy rises after removing spectra with high predicted uncertainty. Export the resulting unified graph in graphml format for Cytoscape visualization and inspect it for cross-ionization-mode clusters, e.g. the caffeine-metabolism cluster in the human urine case study.

## Related tools

- **MS2DeepScore 2.0** (Predicts pairwise chemical (Tanimoto-based) similarities between spectra, including across ionization modes, providing the edge weights for the network) — github.com/matchms/MS2DeepScore
- **matchms Pipeline** (Applies default spectrum filters and runs MS2DeepScore score computations to produce the similarity matrix used for edge selection) — github.com/matchms/ms2deepscore
- **mzmine** (Alternative local deployment combining feature detection with MS2DeepScore-based molecular networking for users without programming expertise) — github.com/matchms/ms2deepscore

## Examples

```
pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); report = pipeline.run(spectrum_file_name); similarity_matrix = pipeline.scores.to_array()
```

## Evaluation signals

- Every retained edge is mutual: each endpoint of an edge appears in the other endpoint's top-20 candidate list
- All edge weights in the final network exceed the 0.85 similarity cutoff and each node contributes at most 20 candidate links before mutuality filtering
- Exported graphml parses correctly and imports into Cytoscape with cross-ionization-mode edges present, not just within-mode clusters
- Known metabolically relevant clusters are recovered — e.g. the caffeine-related cluster links spectra from both modes consistent with known caffeine metabolism pathways
- Spectra flagged by the embedding evaluator with high predicted MSE or low information content are absent from the network

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to the training data
- There remains a risk that the model is not well-suited for very specific tasks or chemical classes, which would propagate into network edges
- The training pair sampling did not enforce balanced sampling of ionization mode pairs, which may have limited negative-vs-negative mode prediction performance

## Evidence

- [intro] unified molecular network from predicted similarities: "By using these predicted similarities by MS2DeepScore, both positive and negative ionization mode spectra can be visualized in one unified molecular network"
- [intro] filter unreliable spectra before trusting predictions: "This allows users to filter out spectra for which the MS2DeepScore predictions are unreliable, e.g., due to low spectral quality or when spectra differ substantially from the training data."
- [discussion] removing high-uncertainty spectra raises accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [readme] similarity matrix is the network construction input: [evidence span withheld — anchor status: external_doc]
- [discussion] cross-mode networking is feasible with this model: "Similarity- based graphs can now be generated independent of the ionization mode, rendering cross-ionization mode molecular networking feasible"
