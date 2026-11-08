---
name: spectral-embedding-similarity-networking
description: Use when you need to organize repository-scale collections of unannotated
  tandem mass spectra (e.g. the 201 million positive-mode MS/MS spectra in MassIVE
  GNPS) into a similarity network that enables propagation of annotations from spectral-library
  nodes to unknowns.
when_to_use_negative:
- When spectra are outside the positive-ionization, singly-charged precursor regime
  (e.g. negative ionization mode or multiply charged precursors), which DreaMS does
  not model.
- When spectra already have reference library matches, direct spectral-library retrieval
  outperforms network-based annotation and the network adds no benefit.
- When the goal is MS1 isotopic-pattern or adduct-distribution analysis; this skill
  ignores those features entirely.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: PyNNDescent
  role: Implements NN-Descent for approximate 3-nearest-neighbor graph construction
    over DreaMS embeddings using cosine similarity
  repo: https://github.com/lmcinnes/pynndescent
- name: DreaMS
  role: Produces the 1,024-dimensional embeddings whose cosine similarity defines
    edge weights; also supplies property/fluorine predictions for node annotation
  repo: https://github.com/pluskal-lab/DreaMS
- name: HDF5
  role: Storage format for the compact binary representation of the molecular network
    (nodes, edges, similarity scores)
- name: MoNA
  role: Reference spectral-library source providing annotated nodes used as anchors
    for shortest-path statistics
- name: NIST20
  role: Reference spectral-library source providing annotated nodes used as anchors
    for shortest-path statistics
- name: GeMS (GNPS Experimental Mass Spectra)
  role: Source corpus of spectra mined from MassIVE whose embeddings form the graph
    nodes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:b80918f011099f136ef85b25517215d57fea8c9a9424fc7c2556c66022f159c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lmcinnes/pynndescent
  tools:
  - DreaMS
  - GeMS (GNPS Experimental Mass Spectra)
  - HDF5
  - MoNA
  - NIST20
  - PyNNDescent
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spectral-embedding-similarity-networking

## Summary

Constructs a molecular network by embedding MS/MS spectra into a 1,024-dimensional DreaMS representation space and building an approximate 3-NN graph over 34 million spectra that represent 201 million GeMS-C spectra, then annotating nodes with predicted properties and library metadata.

## When to use

Use when you need to organize repository-scale collections of unannotated tandem mass spectra (e.g. the 201 million positive-mode MS/MS spectra in MassIVE GNPS) into a similarity network that enables propagation of annotations from spectral-library nodes to unknowns.

## When NOT to use

- When spectra are outside the positive-ionization, singly-charged precursor regime (e.g. negative ionization mode or multiply charged precursors), which DreaMS does not model.
- When spectra already have reference library matches, direct spectral-library retrieval outperforms network-based annotation and the network adds no benefit.
- When the goal is MS1 isotopic-pattern or adduct-distribution analysis; this skill ignores those features entirely.

## Inputs

- Pre-computed DreaMS embeddings (1,024-dimensional real-valued vectors) for GeMS-C1 spectra, stored in the compact HDF5-based binary format designed for deep learning
- DreaMS embeddings for 79,000 merged MoNA and NIST20 spectral-library spectra
- DreaMS property and fluorine predictions (e.g. fluorine detection precision 0.91 at recall 0.57)
- MassIVE GNPS repository metadata (LC-MS/MS experiment/sample descriptions)

## Outputs

- DreaMS Atlas molecular network: approximate 3-NN graph of 34 million nodes representing 201 million MS/MS spectra, stored in the compact HDF5-based binary format
- Edges annotated with cosine-similarity scores in the DreaMS embedding space
- Annotated nodes: MassIVE metadata, DreaMS-predicted molecular properties and fluorine content, and MoNA/NIST20 library annotations
- Atlas statistics (e.g. fraction of edges with similarity > 0.8, connected-component size, median shortest path to library nodes)

## How to apply

Load pre-computed DreaMS embeddings (1,024-dimensional real-valued vectors) for the 76 million GeMS-C1 spectra plus 79,000 merged MoNA/NIST20 library spectra from the compact HDF5-based binary format. Run NN-Descent (PyNNDescent) to build an approximate 3-nearest-neighbor graph over the embedding vectors using cosine similarity, since zero-shot cosine similarity in the embedding space outperforms MS2DeepScore. Collapse neighborhoods of highly similar spectra into single nodes by iteratively merging all edges with similarity greater than 0.9 using a BFS traversal to find connected components of similar spectra. Store the resulting network (nodes, edges, similarity scores) in the HDF5-based binary format, and populate nodes with DreaMS property and fluorine predictions plus MassIVE metadata and MoNA/NIST20 library nodes. The rationale is that this collapses 201 million spectra into 34 million nodes while keeping 67% of edges above similarity 0.8 and keeping 99.7% of nodes in a single connected component.

## Related tools

- **PyNNDescent** (Implements NN-Descent for approximate 3-nearest-neighbor graph construction over DreaMS embeddings using cosine similarity) — https://github.com/lmcinnes/pynndescent
- **DreaMS** (Produces the 1,024-dimensional embeddings whose cosine similarity defines edge weights; also supplies property/fluorine predictions for node annotation) — https://github.com/pluskal-lab/DreaMS
- **HDF5** (Storage format for the compact binary representation of the molecular network (nodes, edges, similarity scores))
- **MoNA** (Reference spectral-library source providing annotated nodes used as anchors for shortest-path statistics)
- **NIST20** (Reference spectral-library source providing annotated nodes used as anchors for shortest-path statistics)
- **GeMS (GNPS Experimental Mass Spectra)** (Source corpus of spectra mined from MassIVE whose embeddings form the graph nodes)

## Examples

```
python -c "from pynndescent import NNDescent; import numpy as np; idx = NNDescent(np.load('embeddings.npy'), n_neighbors=3, metric='cosine'); G, _ = idx.query(np.load('embeddings.npy'), k=3)"
```

## Evaluation signals

- Verify node count is approximately 34 million (representing 201 million spectra after similarity>0.9 merging).
- Confirm 67% of edges have similarity > 0.8.
- Check that 99.7% of nodes form a single connected component.
- Compute the median shortest path from randomly selected nodes to the nearest annotated MoNA/NIST20 library node; it should be six edges.
- Confirm embeddings are 1,024-dimensional real-valued vectors and the network is persisted in the compact HDF5-based binary format.

## Limitations

- Merging threshold of similarity > 0.9 collapses neighborhoods, which may over-cluster chemically distinct but spectrally similar isomers.
- The approximate 3-NN construction (NN-Descent) trades exactness for scalability; some true neighbors may be missed.
- The network covers only positive-mode, singly charged precursor spectra from the MassIVE GNPS repository; it does not extend to other ionization modes or charge states.
- Fine-tuned DreaMS property/fluorine predictions are used for annotation but are not guaranteed to be production-ready for all chemical classes.

## Evidence

- [abstract] Atlas construction method: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [discussion] NN-Descent tool: "the PyNNDescent implementation of NN-Descent by McInnes et al. (https://github.com/lmcinnes/pynndescent)"
- [abstract] Storage format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Embedding similarity outperforms MS2DeepScore: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] Atlas statistics: "99.7% of the nodes form a single connected component"
- [abstract] Median shortest path to library node: "The median length of the shortest path from a randomly selected node to the nearest annotated node from a spectral library is six edges"
