---
name: large-graph-connectivity-statistics-computation
description: Use when you have a large molecular networking edge list (tandem MS nodes
  with cosine-similarity edges) and need to quantify how a network reconstruction
  changes connectivity, e.g. singleton reduction or gain of annotation-amenable nodes.
when_to_use_negative:
- Input is a feature table or raw MS spectra rather than a node/edge molecular similarity
  graph
- The graph is small or already densely connected, so singleton-fraction statistics
  would not distinguish networking strategies
- The goal is spectral library annotation itself rather than evaluating network topology/connectivity
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: molecular_communities
  role: Source codebase implementing molecular community networking (MCN) whose reconstructed
    network the connectivity statistics are computed on
  repo: https://github.com/Alexander0/molecular_communities
- name: networkx
  role: Builds the weighted cosine-similarity graph, runs Louvain community detection,
    and computes per-community maximum-weight spanning trees and connectivity statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:805b6371eda53acd6b0d5a196b8680adc2ecb0ba00029e550e85a640571e9cc5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  - networkx
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# large-graph-connectivity-statistics-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes connectivity statistics (linked-node fraction, singleton fraction, nodes newly connected to annotated nodes) on very large molecular similarity graphs (~8.4M nodes) from GNPS/MassIVE, comparing a conventional cosine-0.7 network against the MCN reconstruction.

## When to use

Use when you have a large molecular networking edge list (tandem MS nodes with cosine-similarity edges) and need to quantify how a network reconstruction changes connectivity, e.g. singleton reduction or gain of annotation-amenable nodes.

## When NOT to use

- Input is a feature table or raw MS spectra rather than a node/edge molecular similarity graph
- The graph is small or already densely connected, so singleton-fraction statistics would not distinguish networking strategies
- The goal is spectral library annotation itself rather than evaluating network topology/connectivity

## Inputs

- GNPS/MassIVE global molecular network edge list (node pairs with cosine similarity scores)
- Node table with tandem MS spectra and library-annotation (identified) flags for ~8.4M molecular nodes
- Conventional molecular network built with cosine-0.7 similarity threshold (baseline for comparison)

## Outputs

- Reconstructed MCN edge list (union of per-community maximum-weight spanning trees)
- mcn_network_statistics.csv with linked-node fraction, singleton fraction, and count of additional nodes connected to an annotated node

## How to apply

Download the public GNPS/MassIVE global molecular network (nodes with tandem MS spectra, edges with cosine similarity scores) and load the edge list into a networkx weighted graph. Run Louvain community detection to partition the ~8.4M molecular nodes into communities, then within each Louvain community compute a maximum-weight spanning tree over the cosine-similarity edges and union the per-community trees to form the reconstructed MCN. Compute the fraction of linked nodes and the singleton fraction of the MCN, and count nodes newly connected to an annotated (library-identified) node relative to the conventional cosine-0.7 network. Save the reconstructed MCN edge list and a statistics table (mcn_network_statistics.csv); this matters because conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, while MCN optimizes connectivity per metabolite.

## Related tools

- **molecular_communities** (Source codebase implementing molecular community networking (MCN) whose reconstructed network the connectivity statistics are computed on) — https://github.com/Alexander0/molecular_communities
- **networkx** (Builds the weighted cosine-similarity graph, runs Louvain community detection, and computes per-community maximum-weight spanning trees and connectivity statistics)

## Examples

```
import networkx as nx; G = nx.from_pandas_edgelist(edges, 'node1', 'node2', edge_attr='cosine'); comms = nx.community.louvain_communities(G, weight='cosine')
```

## Evaluation signals

- Singleton fraction drops from 57.4% in the conventional cosine-0.7 network to 4.9% in the MCN
- Approximately 95% (~8M of ~8.4M) of nodes are linked in the reconstructed MCN
- Approximately 3.5 million additional nodes are connected to an annotated (library-identified) node relative to the conventional network
- mcn_network_statistics.csv exists and its reproduced values match the reported 57.4% / 4.9% / ~3.5M numbers
- Every node appears in exactly one Louvain community and each per-community spanning tree is acyclic before unioning

## Limitations

- Statistics are benchmarked against the conventional network's arbitrary cosine-0.7 similarity threshold, which can fracture natural metabolite families, so results depend on that baseline choice
- The repository README is empty (no changelog or operational documentation), so implementation details must come from the workflow description rather than repo docs

## Evidence

- [abstract] Conventional molecular networking uses arbitrary similarity thresholds that can fracture natural metabolite families: [evidence span withheld — anchor status: not_found]
- [abstract] Molecular community networking (MCN) optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise hidden metabolite connections: [evidence span withheld — anchor status: not_found]
- [abstract] Nature's molecular diversity is not random but displays intricate organization stemming from biological necessity: [evidence span withheld — anchor status: not_found]
