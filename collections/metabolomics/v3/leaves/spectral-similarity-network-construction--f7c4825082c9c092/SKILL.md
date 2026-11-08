---
name: spectral-similarity-network-construction
description: Use when you have a collection of tandem MS spectra (e.g., a GNPS/MassIVE
  library or dataset edge list with cosine similarity scores) and need an unpruned
  weighted similarity graph as the substrate for community detection and tree-based
  network reconstruction.
when_to_use_negative:
- The analysis goal is a conventional molecular network that hard-cuts edges at a
  cosine threshold (e.g., 0.7) — this skill deliberately retains all pairwise edges
  and would not reproduce that workflow
- The input is already a per-community subgraph of a completed MCN — the appropriate
  step there is maximum-weight spanning-tree pruning, not full network construction
- You need a predefined number of clusters; MCN/Louvain derives community structure
  from modularity optimization rather than preset community sizes or counts
edam_operation: http://edamontology.org/operation_0292
tools:
- name: NetworkX
  role: Builds the full-connectivity weighted similarity graph, partitions it per
    community labels, and computes maximum_spanning_tree with the spectral similarity
    score as edge weight
- name: pandas
  role: Loads the spectral-similarity edge list and Louvain community assignment table
    and writes node-to-community and statistics tables (communities.csv, mcn_network_statistics.csv)
- name: Louvain (python-louvain / NetworkX community detection)
  role: Modularity-optimization community detection on the unpruned network, with
    no predefined number or sizes of communities
- name: molecular_communities
  role: Deposited repository supplying the reference spectra collection and network
    data used to seed and validate the reconstructed MCN
  repo: Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:63d0fb13b1067749ce138f21542855574a8e999801da24a8a6679156e9cfd1a5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Louvain (python-louvain / NetworkX community detection)
  - NetworkX
  - molecular_communities
  - pandas
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spectral-similarity-network-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Constructs a full-connectivity molecular similarity network from pairwise spectral similarity scores (cosine between tandem MS spectra) with no arbitrary similarity threshold, so that downstream molecular community networking (MCN) can optimize connectivity for each metabolite instead of fracturing natural metabolite families at a fixed cosine cutoff.

## When to use

Use when you have a collection of tandem MS spectra (e.g., a GNPS/MassIVE library or dataset edge list with cosine similarity scores) and need an unpruned weighted similarity graph as the substrate for community detection and tree-based network reconstruction.

## When NOT to use

- The analysis goal is a conventional molecular network that hard-cuts edges at a cosine threshold (e.g., 0.7) — this skill deliberately retains all pairwise edges and would not reproduce that workflow
- The input is already a per-community subgraph of a completed MCN — the appropriate step there is maximum-weight spanning-tree pruning, not full network construction
- You need a predefined number of clusters; MCN/Louvain derives community structure from modularity optimization rather than preset community sizes or counts

## Inputs

- Spectral-similarity weighted edge list (pairs of metabolite/spectrum nodes with cosine similarity scores as edge weights)
- Louvain community assignment table (communities.csv, one row per metabolite node with its community identifier)
- Reference spectra collection (e.g., the cond_mcn_reference_spectra condition from the molecular_communities repository)

## Outputs

- Unpruned full-connectivity weighted similarity graph (networkx Graph)
- Node-to-community mapping (communities.csv)
- Per-community maximum-weight spanning trees exported as community_spanning_trees.graphml
- Edge list with per-node community/cluster assignments (e.g., mcn_reference_spectra_network.csv)
- Statistics table (mcn_network_statistics.csv) with singleton fraction and linked-node fraction

## How to apply

Load the spectral-similarity weighted edge list (e.g., deposited reference spectra from the Alexander0/molecular_communities repository) and build a weighted graph in networkx, retaining ALL pairwise edges — unlike conventional molecular networking, no cosine similarity threshold such as the conventional 0.7 cutoff is applied, because arbitrary thresholds can fracture natural metabolite families and discard connectivity information. If starting from raw spectra, compute pairwise cosine spectral similarity between all spectra to seed the edge list (e.g., mcn_reference_spectra_network.csv pairs per-node community/cluster assignments with edges). Partition the unpruned graph with the Louvain method, letting modularity optimization determine community structure rather than presetting the number or size of communities. Finally, prune for interpretability by computing nx.maximum_spanning_tree per community with the spectral similarity score as edge weight, so only the strongest similarities are kept while every node in the community stays connected (edge count equals node count minus 1). The rationale is that MCN optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise hidden metabolite connections.

## Related tools

- **NetworkX** (Builds the full-connectivity weighted similarity graph, partitions it per community labels, and computes maximum_spanning_tree with the spectral similarity score as edge weight)
- **pandas** (Loads the spectral-similarity edge list and Louvain community assignment table and writes node-to-community and statistics tables (communities.csv, mcn_network_statistics.csv))
- **Louvain (python-louvain / NetworkX community detection)** (Modularity-optimization community detection on the unpruned network, with no predefined number or sizes of communities)
- **molecular_communities** (Deposited repository supplying the reference spectra collection and network data used to seed and validate the reconstructed MCN) — Alexander0/molecular_communities

## Examples

```
import networkx as nx; T = nx.maximum_spanning_tree(G, weight='cosine_similarity')
```

## Evaluation signals

- Each exported community subnetwork is a spanning tree of its parent community: edge count equals node count minus 1
- Every exported subnetwork contains every node assigned to that community by Louvain (no nodes dropped during pruning)
- communities.csv contains exactly one community assignment per network node, and the number of communities emerged from modularity optimization rather than being predefined
- On the reference-spectra benchmark, MCN-derived clusters group spectra of structurally related molecules together (qualitative check against Figures 1g-i and S2-S11)
- On the global GNPS/MassIVE network (~8.4M nodes), reproduced statistics match ~95% (~8M) of nodes linked, singleton fraction reduced from 57.4% (conventional cosine-0.7 network) to 4.9% (MCN), and ~3.5M additional nodes connected to an annotated node

## Limitations

- Applying an arbitrary similarity threshold (e.g., conventional cosine 0.7) can fracture natural metabolite families and create spurious singletons; the full-connectivity construction exists precisely to avoid this, so introducing a cutoff mid-pipeline undermines the method
- Singleton-reduction gains are relative to a conventional cosine-0.7 network; on datasets where thresholded networks already recover most relationships, the reported improvements (57.4% to 4.9% singletons) may not transfer
- No changelog was found for the associated repository, so version-specific behavior of the deposited code is unverified

## Evidence

- [abstract] Organization of molecular diversity motivates the network structure: [evidence span withheld — anchor status: not_found]
