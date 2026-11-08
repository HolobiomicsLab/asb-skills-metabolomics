---
name: graph-shortest-path-median-analysis
description: 'Use when characterizing an embedding-based molecular network: you need
  a single scalar metric describing how many graph edges separate the bulk of unannotated
  spectra from reference-library nodes, to validate that the network''s structure
  reflects annotation coverage.'
when_to_use_negative:
- Input is a raw unweighted spectral similarity matrix (e.g., pairwise cosine scores
  from MS2DeepScore or spectral entropy) without an explicit graph structure — compute
  the 3-NN graph first before running shortest-path statistics.
- You need per-spectrum annotation confidence rather than a network-level connectivity
  metric; shortest-path distance to library nodes does not indicate whether any individual
  spectrum can be structurally annotated.
edam_operation: http://edamontology.org/operation_0573
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3314
tools:
- name: DreaMS
  role: Provides the 1,024-dimensional embedding vectors from which the Atlas 3-NN
    graph and shortest-path statistics are computed; also supplies property and fluorine
    predictions attached to Atlas nodes
  repo: https://github.com/pluskal-lab/DreaMS
- name: PyNNDescent
  role: Approximate nearest-neighbor search (NN-Descent) used to construct the 3-NN
    graph with cosine similarity as the distance metric over DreaMS embeddings
  repo: https://github.com/lmcinnes/pynndescent
- name: GeMS (GNPS Experimental Mass Spectra)
  role: Source dataset of experimental LC-MS/MS spectra (mined from MassIVE GNPS)
    whose DreaMS embeddings populate the Atlas nodes; stored in compact HDF5-based
    binary format designed for deep learning
  repo: https://github.com/pluskal-lab/DreaMS
- name: MoNA
  role: Spectral library whose annotated spectra are merged into Atlas nodes, serving
    as the nearest-annotated-node targets for shortest-path computation
- name: NIST20
  role: Spectral library whose annotated spectra are merged into Atlas nodes, serving
    as the nearest-annotated-node targets for shortest-path computation
- name: HDF5
  role: Storage backend for the compact HDF5-based binary format designed for deep
    learning in which the Atlas graph and its statistics are persisted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:027a8905e48d6eeea2e81d08c9092da870b6f04dd451f1c78e4c8b559fbf92a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - GeMS (GNPS Experimental Mass Spectra)
  - HDF5
  - MoNA
  - NIST20
  - PyNNDescent
  edam_operation: http://edamontology.org/operation_0573
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# graph-shortest-path-median-analysis

## Summary

Computes the median shortest-path distance from randomly sampled nodes to the nearest annotated spectral-library node in the DreaMS Atlas, a 3-NN graph built from DreaMS embeddings of GeMS mass spectra. This statistic quantifies how far, on average, unannotated experimental spectra sit from annotated MoNA/NIST20 reference spectra in the learned embedding space.

## When to use

Use when characterizing an embedding-based molecular network: you need a single scalar metric describing how many graph edges separate the bulk of unannotated spectra from reference-library nodes, to validate that the network's structure reflects annotation coverage.

## When NOT to use

- Input is a raw unweighted spectral similarity matrix (e.g., pairwise cosine scores from MS2DeepScore or spectral entropy) without an explicit graph structure — compute the 3-NN graph first before running shortest-path statistics.
- You need per-spectrum annotation confidence rather than a network-level connectivity metric; shortest-path distance to library nodes does not indicate whether any individual spectrum can be structurally annotated.

## Inputs

- DreaMS Atlas graph in HDF5-based binary format (nodes, edges, and similarity scores), built as a 3-NN graph from 1,024-dimensional DreaMS embeddings of 76 million GeMS-C1 spectra plus ~79,000 merged MoNA/NIST20 library spectra
- Node metadata distinguishing spectral-library (annotated) nodes from GeMS experimental-spectra nodes

## Outputs

- Median shortest-path length (in graph edges) from randomly selected nodes to the nearest annotated spectral-library node
- Optionally, complementary Atlas statistics: fraction of edges with similarity > 0.8, and fraction of nodes in the largest connected component

## How to apply

Load the DreaMS Atlas from its HDF5-based binary format — an approximate 3-NN graph over ~34 million nodes (representing 201 million GeMS-C spectra via LSH and DreaMS clustering) plus ~79,000 merged MoNA/NIST20 library nodes, with edges carrying cosine similarities of DreaMS embeddings. Mark nodes originating from the spectral libraries (MoNA and NIST20) as annotated targets; they are merged into single nodes by collapsing all edges with similarity greater than 0.9 via BFS over highly similar spectra. Randomly sample nodes, compute the shortest path (in edge count) from each sampled node to the nearest library node, and report the median across samples. The article reports a median of six edges, alongside complementary Atlas statistics: 67% of edges with similarity > 0.8 and 99.7% of nodes in a single connected component. Use PyNNDescent with cosine similarity as the distance metric if rebuilding the graph rather than loading a precomputed one.

## Related tools

- **DreaMS** (Provides the 1,024-dimensional embedding vectors from which the Atlas 3-NN graph and shortest-path statistics are computed; also supplies property and fluorine predictions attached to Atlas nodes) — https://github.com/pluskal-lab/DreaMS
- **PyNNDescent** (Approximate nearest-neighbor search (NN-Descent) used to construct the 3-NN graph with cosine similarity as the distance metric over DreaMS embeddings) — https://github.com/lmcinnes/pynndescent
- **GeMS (GNPS Experimental Mass Spectra)** (Source dataset of experimental LC-MS/MS spectra (mined from MassIVE GNPS) whose DreaMS embeddings populate the Atlas nodes; stored in compact HDF5-based binary format designed for deep learning) — https://github.com/pluskal-lab/DreaMS
- **MoNA** (Spectral library whose annotated spectra are merged into Atlas nodes, serving as the nearest-annotated-node targets for shortest-path computation)
- **NIST20** (Spectral library whose annotated spectra are merged into Atlas nodes, serving as the nearest-annotated-node targets for shortest-path computation)
- **HDF5** (Storage backend for the compact HDF5-based binary format designed for deep learning in which the Atlas graph and its statistics are persisted)

## Examples

```
from dreams.atlas import Atlas; A = Atlas.load('atlas.h5'); median_path = A.median_shortest_path_to_library(n_samples=1000)
```

## Evaluation signals

- The reported median shortest path from a randomly selected Atlas node to the nearest annotated spectral-library node is six edges; reproduced values should match this.
- 99.7% of Atlas nodes form a single connected component — verify connectivity before interpreting path statistics, since unreachable nodes would skew the metric.
- 67% of Atlas edges have cosine similarity > 0.8; edges below this threshold contribute to longer shortest paths.
- The graph should contain ~34 million nodes representing 201 million GeMS-C spectra after merging edges with similarity > 0.9 via BFS; confirm node count is in this range before computing statistics.
- Library nodes are the merge of ~79,000 MoNA and NIST20 spectra — the count of annotated targets should be consistent with this scale.

## Limitations

- The Atlas merges nodes with similarity > 0.9, collapsing similar spectra into single nodes — this aggregation changes the effective graph topology and means shortest-path lengths are measured between merged clusters, not individual spectra.
- Only a subset of available mass spectra was used for training and only positive-mode, singly charged precursor spectra were included; the Atlas does not cover negative ionization mode or multiply charged species, limiting the generalization of the shortest-path statistic to those regimes.
- The Atlas disregards MS1 isotopic patterns and adduct distributions, so shortest-path distance does not capture chemical-context information that might otherwise bring nodes closer to annotated references.

## Evidence

- [abstract] The median length of the shortest path from a randomly selected node to the nearest annotated node from a spectral library is six edges: "The median length of the shortest path from a randomly selected node to the nearest annotated node from a spectral library is six edges"
- [abstract] 99.7% of the nodes form a single connected component: "99.7% of the nodes form a single connected component"
- [abstract] The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [discussion] the PyNNDescent implementation of NN-Descent by McInnes et al. (https://github.com/lmcinnes/pynndescent): "the PyNNDescent implementation of NN-Descent by McInnes et al. (https://github.com/lmcinnes/pynndescent)"
- [abstract] the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules: "the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules"
- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
