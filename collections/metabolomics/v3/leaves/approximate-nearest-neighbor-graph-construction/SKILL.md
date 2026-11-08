---
name: approximate-nearest-neighbor-graph-construction
description: Use when you have a large corpus of spectral embeddings (e.g. 1,024-d
  DreaMS vectors) and need a scalable similarity graph for annotation propagation
  or atlas construction.
when_to_use_negative:
- The spectra are raw peak lists rather than pre-trained DreaMS embeddings — embeddings
  must exist first, since the graph is built over vector representations, not spectra
  directly
- You need an exact k-NN graph for small datasets (<100k spectra) where exact search
  is feasible and approximation error matters
- Spectra are from negative ionization mode or multiply charged precursors, which
  DreaMS was not trained on and whose embeddings may be unreliable
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
tools:
- name: PyNNDescent
  role: Implements NN-Descent to build the approximate 3-nearest-neighbor graph over
    1,024-dimensional DreaMS embeddings using cosine similarity
  repo: https://github.com/lmcinnes/pynndescent
- name: DreaMS
  role: Provides the 1,024-dimensional self-supervised spectral embeddings over which
    the NN graph is constructed, and its annotations populate the graph nodes
  repo: https://github.com/pluskal-lab/DreaMS
- name: HDF5
  role: Binary storage format used to store the GeMS spectra and the resulting molecular
    network (nodes, edges, similarity scores)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:a9ad1ae21e636d451dc5e105b4cdbd502ae5cd58f2f6d12f4e0a34344f6c2dfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lmcinnes/pynndescent
  tools:
  - DreaMS
  - HDF5
  - MoNA
  - NIST20
  - PyNNDescent
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# approximate-nearest-neighbor-graph-construction

## Summary

Builds a molecular network by running NN-Descent over 1,024-dimensional DreaMS spectral embeddings to produce an approximate 3-NN graph, then collapsing highly similar spectra into single nodes. Used to turn the 76-million-spectrum GeMS repository into the 34-million-node DreaMS Atlas.

## When to use

Use when you have a large corpus of spectral embeddings (e.g. 1,024-d DreaMS vectors) and need a scalable similarity graph for annotation propagation or atlas construction.

## When NOT to use

- The spectra are raw peak lists rather than pre-trained DreaMS embeddings — embeddings must exist first, since the graph is built over vector representations, not spectra directly
- You need an exact k-NN graph for small datasets (<100k spectra) where exact search is feasible and approximation error matters
- Spectra are from negative ionization mode or multiply charged precursors, which DreaMS was not trained on and whose embeddings may be unreliable

## Inputs

- Pre-computed DreaMS embeddings (1,024-dimensional real-valued vectors) stored in the HDF5-based binary format for ~76 million GeMS-C1 spectra
- Merged MoNA and NIST20 library spectra (~79,000) with annotations to serve as seed/anchor nodes
- Choice of k (k=3) and edge-merge similarity threshold (>0.9)

## Outputs

- Approximate 3-NN molecular network graph stored in the HDF5-based binary format, containing nodes, edges, and cosine similarity scores
- DreaMS Atlas with ~34 million nodes representing 201 million MassIVE GNPS spectra, populated with DreaMS property and fluorine predictions plus MassIVE metadata
- Graph statistics: node count, fraction of edges with similarity > 0.8, largest connected component fraction, median shortest path to nearest spectral-library node

## How to apply

Load DreaMS embedding vectors — 1,024-dimensional real-valued vectors — from the compact HDF5-based binary format designed for deep learning (76 million GeMS-C1 spectra plus ~79,000 merged MoNA/NIST20 library spectra). Run NN-Descent via the PyNNDescent implementation to build an approximate 3-nearest-neighbor graph using cosine similarity as the distance metric, which the paper shows outperforms supervised MS2DeepScore even before fine-tuning. Collapse neighborhoods of highly similar spectra into single nodes by iteratively merging edges with similarity greater than 0.9 using a BFS traversal to find connected components. Store the resulting network (nodes, edges, similarity scores) in the HDF5-based format, then compute statistics (fraction of edges with similarity > 0.8, connectivity fraction, median shortest path to nearest annotated library node) to validate the graph. The rationale: approximate methods are necessary because exact k-NN at repository scale (201 million spectra) is computationally infeasible, and library nodes provide reachable anchors for annotation.

## Related tools

- **PyNNDescent** (Implements NN-Descent to build the approximate 3-nearest-neighbor graph over 1,024-dimensional DreaMS embeddings using cosine similarity) — https://github.com/lmcinnes/pynndescent
- **DreaMS** (Provides the 1,024-dimensional self-supervised spectral embeddings over which the NN graph is constructed, and its annotations populate the graph nodes) — https://github.com/pluskal-lab/DreaMS
- **HDF5** (Binary storage format used to store the GeMS spectra and the resulting molecular network (nodes, edges, similarity scores))

## Examples

```
from pynndescent import NNDescent; index = NNDescent(embeddings, n_neighbors=3, metric='cosine'); graph, _ = index.neighbor_graph
```

## Evaluation signals

- Node count matches ~34 million after edge merging, representing 201 million spectra
- 67% of graph edges have cosine similarity > 0.8
- 99.7% of nodes form a single connected component
- Median shortest path from a randomly selected node to the nearest annotated spectral-library node is six edges
- Edge merging converged: no remaining edges with similarity > 0.9 after BFS-based collapse of connected components of highly similar spectra

## Limitations

- Graph statistics and thresholds (3-NN, similarity > 0.9 for merging, >0.8 for statistics) were calibrated on positive-mode, singly charged metabolomics spectra; performance on other regimes is unvalidated
- DreaMS embeddings disregard MS1 isotopic patterns and adduct distributions, so graph structure may miss chemically distinct species with similar tandem spectra
- The Atlas covers 'virtually all positive-mode metabolomics spectra' but not negative-mode or charged spectra, limiting generalization
- Median path length of six edges means annotation propagation can span multiple hops, compounding error in downstream annotation tasks

## Evidence

- [discussion] Atlas built via PyNNDescent over DreaMS embeddings: "the PyNNDescent implementation of NN-Descent by McInnes et al. (https://github.com/lmcinnes/pynndescent)"
- [abstract] DreaMS Atlas 3-NN graph construction: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [abstract] 99.7% connectivity of Atlas graph: "99.7% of the nodes form a single connected component"
- [abstract] HDF5 storage of spectra and network: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Embedding dimensionality and organization: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [abstract] Zero-shot cosine similarity outperforms MS2DeepScore: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] Spectral library annotations in Atlas: "The median length of the shortest path from a randomly selected node to the nearest annotated node from a spectral library is six edges"
