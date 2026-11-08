---
name: bfs-neighborhood-collapsing
description: Decision support for building a compact molecular network from dense
  3-NN graphs of MS/MS embeddings when highly similar spectra (similarity > 0.9) should
  be merged into single nodes to reduce redundancy before annotation propagation.
when_to_use_negative:
- Input graph is a fully annotated spectral library graph (e.g., MoNA/NIST20 nodes)
  where merging would conflate distinct reference molecules rather than remove redundant
  unannotated spectra
- Spectra were already de-duplicated upstream via LSH clustering with per-cluster
  sampling (e.g., 10 or 1,000 spectra per cluster) so no collapsing is needed
- The graph uses a similarity metric other than cosine on DreaMS embeddings and no
  equivalent 0.9 cutoff has been validated
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: DreaMS
  role: Produces the 1,024-dimensional real-valued embeddings on which the 3-NN graph
    and BFS collapsing are performed
  repo: https://github.com/pluskal-lab/DreaMS
- name: PyNNDescent
  role: Builds the approximate 3-nearest-neighbor graph over the embedding vectors
    before BFS component merging
  repo: https://github.com/lmcinnes/pynndescent
- name: GeMS (GNPS Experimental Mass Spectra)
  role: Source spectra whose LSH clusters and redundant neighborhoods are collapsed
    into single Atlas nodes
- name: MassIVE GNPS repository
  role: Repository of LC-MS/MS experiments mined to obtain the spectra represented
    in the collapsed network
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:b8e62ae402eec1de16676ee6803a41d9e4a9c153bf30f3167fe12ff6897d749a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - GeMS (GNPS Experimental Mass Spectra)
  - MassIVE GNPS repository
  - MoNA
  - NIST20
  - PyNNDescent
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# BFS neighborhood collapsing

## Summary

Collapse near-duplicate spectra in an approximate 3-nearest-neighbor graph into single molecular-network nodes by merging all edges with similarity greater than 0.9 via breadth-first-search traversal of connected components, as used to build the DreaMS Atlas.

## When to use

Decision support for building a compact molecular network from dense 3-NN graphs of MS/MS embeddings when highly similar spectra (similarity > 0.9) should be merged into single nodes to reduce redundancy before annotation propagation.

## When NOT to use

- Input graph is a fully annotated spectral library graph (e.g., MoNA/NIST20 nodes) where merging would conflate distinct reference molecules rather than remove redundant unannotated spectra
- Spectra were already de-duplicated upstream via LSH clustering with per-cluster sampling (e.g., 10 or 1,000 spectra per cluster) so no collapsing is needed
- The graph uses a similarity metric other than cosine on DreaMS embeddings and no equivalent 0.9 cutoff has been validated

## Inputs

- 3-nearest-neighbor graph of MS/MS spectra (nodes = spectra, edges = cosine similarity of DreaMS 1,024-dimensional embeddings) from PyNNDescent
- Similarity scores on edges (e.g., DreaMS cosine similarity)
- Similarity merge threshold (0.9 in the DreaMS Atlas construction)

## Outputs

- Collapsed molecular network with near-duplicate spectra merged into single nodes
- Node, edge, and similarity-score records stored in the compact HDF5-based binary format for deep learning
- Resulting DreaMS Atlas graph amenable to graph statistics (fraction of edges with similarity > 0.8, largest connected component size)

## How to apply

After constructing an approximate 3-NN graph (e.g., via PyNNDescent NN-Descent with cosine similarity over DreaMS 1,024-dimensional embeddings), run a breadth-first search (BFS) to find connected components of the subgraph containing only edges with similarity above the 0.9 threshold, and merge each such component into a single node. Rationale: 34 million representative spectra stand in for 201 million GeMS-C spectra because LSH clustering and this collapsing step remove near-duplicate spectra, so the resulting network is sparse enough to populate with annotations. Store the collapsed nodes, edges, and similarity scores in the compact HDF5-based binary format designed for deep learning. The threshold of 0.9 defines which neighborhood edges indicate the same molecular entity; lower it to keep finer-grained nodes, raise it to merge more aggressively.

## Related tools

- **DreaMS** (Produces the 1,024-dimensional real-valued embeddings on which the 3-NN graph and BFS collapsing are performed) — https://github.com/pluskal-lab/DreaMS
- **PyNNDescent** (Builds the approximate 3-nearest-neighbor graph over the embedding vectors before BFS component merging) — https://github.com/lmcinnes/pynndescent
- **GeMS (GNPS Experimental Mass Spectra)** (Source spectra whose LSH clusters and redundant neighborhoods are collapsed into single Atlas nodes)
- **MassIVE GNPS repository** (Repository of LC-MS/MS experiments mined to obtain the spectra represented in the collapsed network)

## Examples

```
from pynndescent import NNDescent; index = NNDescent(ems, n_neighbors=3, metric='cosine'); graph = index.neighbor_graph; # BFS-collapse edges with similarity > 0.9 into single nodes
```

## Evaluation signals

- All edges with similarity > 0.9 are merged so that every remaining node's neighbors have similarity ≤ 0.9 (no edge above threshold survives within a node)
- Collapsing reduces the graph from the 76 million GeMS-C1 spectra down to the ~34 million Atlas nodes, consistent with reported node counts
- Collapsed graph retains 99.7% of nodes in a single connected component
- 67% of Atlas edges have similarity > 0.8 after collapsing
- Median shortest path from a random node to the nearest annotated spectral-library node equals six edges

## Limitations

- The 0.9 similarity cutoff is calibrated for DreaMS cosine similarity on GeMS-C spectra; applying it to other embeddings or similarity measures may merge distinct molecules or leave redundant nodes
- BFS-based merging is greedy over connected components, so ordering or chained near-duplicates can produce nodes spanning borderline similarity values
- The resulting network depends on upstream LSH clustering and per-cluster random sampling limits (10 or 1,000 spectra), which constrain which duplicates remain to be collapsed

## Evidence

- [methods] Collapse neighborhoods of similar spectra into single nodes by iteratively merging all edges with similarity greater than 0.9 using a breadth-first search (BFS) traversal to find connected components of highly similar spectra.: [evidence span withheld — anchor status: not_found]
- [results] The DreaMS Atlas is built as an approximate 3-NN graph over 34 million spectra (representing 201 million GeMS-C spectra via LSH and DreaMS clustering), populated with DreaMS property and fluorine predictions plus MassIVE metadata and MoNA/NIST20 library nodes.: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [results] 34 million nodes, 67% of edges having similarity > 0.8, 99.7% of nodes forming a single connected component, and a median shortest path of six edges to the nearest spectral-library node.: [evidence span withheld — anchor status: not_found]
- [abstract] DreaMS representations used for the 3-NN graph are 1,024-dimensional real-valued vectors.: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules"
- [readme] PyNNDescent implements NN-Descent for approximate nearest-neighbor graph construction.: [evidence span withheld — anchor status: external_doc]
