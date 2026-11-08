---
name: graphml-network-export-for-cytoscape
description: Use when MS2DeepScore-predicted similarities between tandem mass spectra
  (including cross-ionization-mode pairs) must be turned into a unified molecular
  network .graphml file for Cytoscape visualization and cluster inspection.
when_to_use_negative:
- The similarity matrix has not yet been computed or spectra flagged as unreliable
  (high predicted embedding error / low information content) have not been filtered
  out — network edges would be built on unstable scores
- The goal is one-to-one library annotation of individual spectra (e.g. via MS2Query
  or library matching) rather than cross-mode network exploration
- Analyses restricted to a single ionization mode where standard same-mode cosine
  networking is sufficient and cross-mode linking is not needed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MS2DeepScore 2.0
  role: Computes the predicted chemical similarities (both within and across ionization
    modes) from which top-20 mutual edges above 0.85 are selected for the network
  repo: https://github.com/matchms/MS2DeepScore
- name: matchms
  role: Provides the cleaning/filtering pipeline (DEFAULT_FILTERS) and Pipeline/score
    object whose similarity matrix feeds network construction
  repo: https://github.com/matchms/ms2deepscore
- name: mzmine
  role: 'Alternative no-code route: integrates MS2DeepScore-based molecular networking
    with feature detection for users without programming expertise'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:cc37f324450890e8e3bd3b19134102c6f90c295516cc87f975235da0208cfbd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore 2.0
  - matchms
  - mzmine
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# graphml-network-export-for-cytoscape

## Summary

Converts an MS2DeepScore-predicted pairwise similarity matrix (spanning positive and negative ionization mode spectra) into a unified molecular network graph and exports it as GraphML for Cytoscape visualization of cross-ionization-mode clusters.

## When to use

Use when MS2DeepScore-predicted similarities between tandem mass spectra (including cross-ionization-mode pairs) must be turned into a unified molecular network .graphml file for Cytoscape visualization and cluster inspection.

## When NOT to use

- The similarity matrix has not yet been computed or spectra flagged as unreliable (high predicted embedding error / low information content) have not been filtered out — network edges would be built on unstable scores
- The goal is one-to-one library annotation of individual spectra (e.g. via MS2Query or library matching) rather than cross-mode network exploration
- Analyses restricted to a single ionization mode where standard same-mode cosine networking is sufficient and cross-mode linking is not needed

## Inputs

- Pairwise MS2DeepScore similarity matrix (numpy array) between all spectra, computed within and across positive/negative ionization modes
- Preprocessed spectra file (mgf, msp, mzml, mzxml, json or usi) with ionmode and precursor_mz metadata
- Edge-construction parameters: top-20 links per spectrum, similarity cutoff 0.85, mutual-link retention

## Outputs

- Unified molecular network graph spanning positive and negative mode spectra as nodes with MS2DeepScore-scored edges
- GraphML (.graphml) file loadable in Cytoscape

## How to apply

Start from the full pairwise MS2DeepScore similarity matrix computed with the pre-trained model over cleaned spectra from both ionization modes (matchms DEFAULT_FILTERS pipeline). Filter the predicted similarities per spectrum to the top-20 highest-scoring links with similarity above the 0.85 cutoff, then keep only mutual links (edges appearing in both spectra's top-20 lists) to define network edges. Construct the graph so that nodes represent spectra (positive and negative mode together) and edges carry the predicted MS2DeepScore scores as attributes, then export the graph in GraphML format for Cytoscape. The rationale is that MS2DeepScore 2.0 predicts Tanimoto-like chemical similarity between spectra of the same and different ionization modes, so a single unified graph can link biochemically related metabolites across polarity — e.g. the caffeine-metabolism cluster containing spectra from both modes in the human urine case study. Judge correctness by loading the .graphml in Cytoscape and confirming cross-mode clusters are recovered.

## Related tools

- **MS2DeepScore 2.0** (Computes the predicted chemical similarities (both within and across ionization modes) from which top-20 mutual edges above 0.85 are selected for the network) — https://github.com/matchms/MS2DeepScore
- **matchms** (Provides the cleaning/filtering pipeline (DEFAULT_FILTERS) and Pipeline/score object whose similarity matrix feeds network construction) — https://github.com/matchms/ms2deepscore
- **mzmine** (Alternative no-code route: integrates MS2DeepScore-based molecular networking with feature detection for users without programming expertise)

## Examples

```
similarity_matrix = pipeline.scores.to_array(); import networkx as nx; nx.write_graphml(G, "urine_ms2ds_network.graphml")  # G built from mutual top-20 edges with score > 0.85
```

## Evaluation signals

- Every retained edge is mutual: each spectrum appears in the other's top-20 list, and each edge score exceeds the 0.85 cutoff
- The exported .graphml loads in Cytoscape with node count equal to the number of input spectra and edge attributes carrying MS2DeepScore scores in [0, 1]
- The network contains clusters spanning both ionization modes (cross-mode edges), not two disconnected positive-mode and negative-mode components
- Known-biochemistry check: a caffeine-related cluster is recovered containing spectra from both modes, consistent with caffeine metabolism pathways and confirmed by expert annotation
- Spectra flagged by the embedding evaluator as low quality or dissimilar to training data are excluded before edge construction

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra from multiple precursor ions, or spectra dissimilar to the training data, which can create spurious network edges
- The pair sampling algorithm did not enforce balanced sampling across ionization-mode pairs, so negative-vs-negative mode predictions may perform worse than other pair types
- There remains a risk the model is not well-suited for very specific tasks or chemical classes, so cluster annotation should be validated (e.g. expert confirmation as done for the caffeine cluster)

## Evidence

- [intro] Cross-mode similarity computation feeds the network: "By using these predicted similarities by MS2DeepScore, both positive and negative ionization mode spectra can be visualized in one unified molecular network"
- [discussion] Cross-mode graph generation is explicitly enabled: "Similarity- based graphs can now be generated independent of the ionization mode, rendering cross-ionization mode molecular networking feasible"
- [readme] Similarity matrix is the direct input to graph construction: [evidence span withheld — anchor status: external_doc]
- [other] Concrete edge-selection thresholds from the reproduced workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Networked output supports metabolite annotation: "MS2DeepScore enabled cross-ionization mode molecular networking, enhancing data exploration and metabolite annotation"
