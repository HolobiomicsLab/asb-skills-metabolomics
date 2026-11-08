---
name: connected-component-statistics-computation
description: Use when you have built an approximate k-NN molecular network (e.g. the
  DreaMS Atlas 3-NN graph over ~34M spectra) and need to quantify its connectivity
  and distance to annotated nodes.
when_to_use_negative:
- The network is not an approximate k-NN similarity graph (e.g. you have a directed
  reaction network or a taxonomy tree) — edge-similarity and shortest-path-to-library
  statistics are meaningless.
- Spectra lack 1,024-dimensional DreaMS embeddings or equivalent spectral-similarity
  scores, so edge weights cannot be thresholded at 0.8/0.9.
- You need per-molecule annotations rather than graph-level connectivity summaries
  — use the DreaMS annotation tasks (fingerprint prediction, fluorine detection) instead.
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PyNNDescent
  role: Constructs the approximate 3-nearest-neighbor graph from DreaMS embeddings;
    its cosine similarity output is thresholded and merged before statistics are computed
  repo: https://github.com/lmcinnes/pynndescent
- name: DreaMS
  role: Produces the 1,024-dimensional real-valued embeddings over which the 3-NN
    graph and Atlas statistics are computed
- name: HDF5
  role: Compact binary storage format for the GeMS spectra and resulting Atlas network
    from which statistics are read
- name: MoNA
  role: Annotated spectral-library nodes used as reference targets for shortest-path
    distance computation
- name: NIST20
  role: Annotated spectral-library nodes used as reference targets for shortest-path
    distance computation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:2b8cb565e5ce77d6f329d51ecb8ac6f2731cad06fd1c88e3b61e470f9e2854aa
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# connected-component-statistics-computation

## Summary

Computes graph-level statistics — node count, fraction of high-similarity edges, size of the largest connected component, and median shortest path to annotated library nodes — on the DreaMS Atlas 3-NN molecular network stored in an HDF5-based binary format.

## When to use

Use when you have built an approximate k-NN molecular network (e.g. the DreaMS Atlas 3-NN graph over ~34M spectra) and need to quantify its connectivity and distance to annotated nodes.

## When NOT to use

- The network is not an approximate k-NN similarity graph (e.g. you have a directed reaction network or a taxonomy tree) — edge-similarity and shortest-path-to-library statistics are meaningless.
- Spectra lack 1,024-dimensional DreaMS embeddings or equivalent spectral-similarity scores, so edge weights cannot be thresholded at 0.8/0.9.
- You need per-molecule annotations rather than graph-level connectivity summaries — use the DreaMS annotation tasks (fingerprint prediction, fluorine detection) instead.

## Inputs

- HDF5-based binary molecular network file containing nodes, edges, and similarity scores (the DreaMS Atlas format)
- Approximate 3-NN graph built with PyNNDescent over 1,024-dimensional DreaMS embeddings (cosine distance) after merging edges with similarity > 0.9
- Node annotations identifying which nodes correspond to MoNA/NIST20 spectral-library spectra

## Outputs

- Total node count (reported as 34 million nodes for the Atlas)
- Fraction of edges with similarity > 0.8 (reported 67%)
- Fraction of nodes in the largest connected component (reported 99.7%)
- Median shortest path length from randomly selected nodes to the nearest annotated spectral-library node (reported six edges)

## How to apply

Load the molecular network from the compact HDF5-based binary format that stores nodes, edges, and cosine similarity scores produced by PyNNDescent 3-NN graph construction with post-hoc BFS merging of edges above similarity 0.9. Count nodes and edges, then compute the fraction of edges with similarity greater than 0.8 as a high-confidence connectivity proxy. Identify the largest connected component and report the fraction of nodes it contains — the article reports 99.7% of the DreaMS Atlas nodes form a single connected component. Select randomly sampled nodes and compute the median shortest path to the nearest annotated node from MoNA/NIST20 spectral libraries, expecting a median of six edges. These statistics serve both as quality-control validation of the graph-construction pipeline and as a characterization of how far unannotated spectra sit from reference annotations.

## Related tools

- **PyNNDescent** (Constructs the approximate 3-nearest-neighbor graph from DreaMS embeddings; its cosine similarity output is thresholded and merged before statistics are computed) — https://github.com/lmcinnes/pynndescent
- **DreaMS** (Produces the 1,024-dimensional real-valued embeddings over which the 3-NN graph and Atlas statistics are computed)
- **HDF5** (Compact binary storage format for the GeMS spectra and resulting Atlas network from which statistics are read)
- **MoNA** (Annotated spectral-library nodes used as reference targets for shortest-path distance computation)
- **NIST20** (Annotated spectral-library nodes used as reference targets for shortest-path distance computation)

## Examples

```
python -c "import dreams.atlas as A; g=A.load('atlas.h5'); print(len(g.nodes), (g.edge_sim>0.8).mean(), g.largest_cc_frac(), g.median_shortest_path_to_library())"
```

## Evaluation signals

- Node count matches the article's reported 34 million nodes for the DreaMS Atlas
- Fraction of edges with similarity > 0.8 is approximately 67%
- Fraction of nodes in the largest connected component is approximately 99.7% (single dominant component)
- Median shortest path from a randomly sampled node to the nearest MoNA/NIST20 annotated node equals six edges
- Graph file parses cleanly in the HDF5-based binary format without missing node/edge records

## Limitations

- Statistics depend on the upstream 3-NN graph quality — if embeddings are noisy or the 0.9 merge threshold mis-applied, connectivity numbers will diverge from the reported values
- The Atlas covers virtually all positive-mode metabolomics spectra; statistics would not transfer to negative ionization mode or multiply charged precursor spectra, which the article explicitly did not incorporate
- The BFS-based collapse of edges with similarity > 0.9 discards near-duplicate spectra, so node counts reflect merged clusters rather than raw spectra (34M nodes representing 201M spectra)
- Shortest-path-to-library metric is sensitive to which library nodes (MoNA vs NIST20) are considered annotated

## Evidence

- [abstract] 99.7% of the nodes form a single connected component: "99.7% of the nodes form a single connected component"
- [abstract] Median shortest path from a random Atlas node to the nearest spectral library node is six edges: "The median length of the shortest path from a randomly selected node to the nearest annotated node from a spectral library is six edges"
- [abstract] Atlas construction via 3-NN graph over 34 million spectra: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [discussion] PyNNDescent implementation used for NN-Descent: "the PyNNDescent implementation of NN-Descent by McInnes et al."
- [abstract] 1,024-dimensional DreaMS representations organized by structural similarity: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules"
- [abstract] HDF5-based binary format for deep learning storage: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
