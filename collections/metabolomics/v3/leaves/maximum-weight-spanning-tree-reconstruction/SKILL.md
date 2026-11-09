---
name: maximum-weight-spanning-tree-reconstruction
description: Use when a weighted molecular similarity graph built from tandem MS spectra
  edges must be re-connected so that every metabolite keeps its relationship instead
  of being fractured by a fixed cosine-0.7 cutoff.
when_to_use_negative:
- The goal is to reproduce the conventional molecular network defined by a fixed cosine
  similarity threshold (e.g. 0.7) rather than per-community optimized connectivity.
- The input lacks edge weights (cosine similarity scores) or tandem MS spectral nodes,
  since the maximum-weight spanning tree construction depends on weighted cosine edges.
- Only pairwise spectral similarity clustering is needed without any graph-wide re-ordering
  of molecular families.
edam_operation: null
tools:
- name: networkx
  role: Builds the weighted molecular similarity graph, runs Louvain community detection,
    and computes the per-community maximum-weight spanning trees.
- name: molecular_communities
  role: Reference implementation repository for the MCN reconstruction pipeline used
    to reproduce the GNPS/MassIVE global network statistics.
  repo: Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:dfe25b3a757a30f5a2d42f0ec817903a2124330a496c5333ccb23cddbf336223
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  - networkx
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# maximum-weight-spanning-tree-reconstruction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs a molecular community network (MCN) by partitioning a cosine-similarity molecular graph with Louvain community detection and then computing a maximum-weight spanning tree within each community, replacing arbitrary cosine thresholds with per-community optimal connectivity that preserves natural metabolite families.

## When to use

Use when a weighted molecular similarity graph built from tandem MS spectra edges must be re-connected so that every metabolite keeps its best relationship instead of being fractured by a fixed cosine-0.7 cutoff.

## When NOT to use

- The goal is to reproduce the conventional molecular network defined by a fixed cosine similarity threshold (e.g. 0.7) rather than per-community optimized connectivity.
- The input lacks edge weights (cosine similarity scores) or tandem MS spectral nodes, since the maximum-weight spanning tree construction depends on weighted cosine edges.
- Only pairwise spectral similarity clustering is needed without any graph-wide re-ordering of molecular families.

## Inputs

- Public GNPS/MassIVE global molecular network edge list (pairs of molecular nodes with cosine similarity scores as edge weights)
- Node/spectrum table for the ~8.4M molecular nodes with tandem MS spectra and library-identification (annotated-node) flags

## Outputs

- Reconstructed MCN edge list (per-community maximum-weight spanning trees unioned into one graph)
- Statistics table mcn_network_statistics.csv with linked-node fraction, singleton fraction, and count of nodes newly connected to an annotated node

## How to apply

First download the public GNPS/MassIVE global molecular network (nodes carrying tandem MS spectra, edges carrying cosine similarity scores) and load the edge list into a weighted networkx graph over the ~8.4M molecular nodes. Run Louvain community detection to partition nodes into molecular communities. Within each community, compute a maximum-weight spanning tree over the cosine-similarity edges and union the per-community trees to form the reconstructed MCN — this is the key decision point: tree construction is local to each Louvain community, not global, because molecular families are natural communities rather than arbitrary threshold cuts. Compute the linked-node fraction and singleton fraction of the MCN, and count nodes newly connected to an annotated (library-identified) node relative to the conventional cosine-0.7 network. Save the reconstructed MCN edge list plus a statistics table (mcn_network_statistics.csv) and check it against the published values (~95% of nodes linked, singletons reduced from 57.4% to 4.9%, ~3.5M additional annotated-connected nodes).

## Related tools

- **networkx** (Builds the weighted molecular similarity graph, runs Louvain community detection, and computes the per-community maximum-weight spanning trees.)
- **molecular_communities** (Reference implementation repository for the MCN reconstruction pipeline used to reproduce the GNPS/MassIVE global network statistics.) — Alexander0/molecular_communities

## Examples

```
import networkx as nx; G = nx.from_edgelist(edges); comms = nx.community.louvain_communities(G); T = nx.maximum_spanning_tree(G.subgraph(c), weight='cosine')
```

## Evaluation signals

- Singleton fraction drops from 57.4% (conventional cosine-0.7 network) to 4.9% in the reconstructed MCN.
- Approximately 95% of nodes (~8M of ~8.4M) are linked in the MCN.
- About 3.5 million additional nodes become connected to an annotated (library-identified) node compared with the conventional network.
- Each per-community tree is a valid spanning tree: it connects all community members with exactly n-1 edges and is acyclic.
- mcn_network_statistics.csv is produced and its reproduced values match the reported values above.

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families; conversely, the MCN result depends on the Louvain community partition, so errors in community detection propagate into the spanning trees.
- The procedure is validated on the global GNPS/MassIVE network (~8.4M nodes); behavior on much smaller or sparser spectral networks is not characterized in the provided context.
- Comparison against the conventional network requires library-annotated node flags; without annotations the annotated-connectivity gain cannot be computed.

## Evidence

- [abstract] Molecular diversity is organized: [evidence span withheld — anchor status: not_found]
