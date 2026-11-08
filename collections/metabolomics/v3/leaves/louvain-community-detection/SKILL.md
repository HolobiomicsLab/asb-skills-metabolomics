---
name: louvain-community-detection
description: Use when a weighted metabolite similarity network (e.g. GNPS/MassIVE
  cosine-similarity edges) must be partitioned into communities without preset cluster
  counts, to reconstruct MCN structure or quantify singleton reduction.
when_to_use_negative:
- Input network has already been pruned by an arbitrary cosine-similarity threshold
  (e.g. 0.7), since the method requires the full-connectivity network to recover relationships
  that such thresholds fracture
- Analysis requires a predefined number or sizes of communities; Louvain must be run
  without preset community counts
- Input is a raw feature table or spectra rather than a pairwise similarity graph;
  build the weighted network first
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: Louvain
  role: Modularity-optimization community-detection algorithm applied to the unpruned
    metabolite similarity network to produce the molecular communities
- name: networkx
  role: Builds the weighted molecular similarity graph from the GNPS/MassIVE edge
    list and runs Louvain community detection over its ~8.4M nodes
- name: molecular_communities
  role: Source repository providing the full-connectivity metabolite similarity network
    data used for Louvain-based MCN reconstruction
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:bfc9b535aff3ac38849140ed6eaa7959d5d07f76ecb2c793a230804b3cc11eb4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Louvain
  - molecular_communities
  - networkx
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# louvain-community-detection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Applies Louvain modularity optimization to partition metabolite similarity networks into molecular communities, replacing arbitrary cosine-similarity thresholds with data-driven community structure in molecular community networking (MCN).

## When to use

Use when a weighted metabolite similarity network (e.g. GNPS/MassIVE cosine-similarity edges) must be partitioned into communities without preset cluster counts, to reconstruct MCN structure or quantify singleton reduction.

## When NOT to use

- Input network has already been pruned by an arbitrary cosine-similarity threshold (e.g. 0.7), since the method requires the full-connectivity network to recover relationships that such thresholds fracture
- Analysis requires a predefined number or sizes of communities; Louvain must be run without preset community counts
- Input is a raw feature table or spectra rather than a pairwise similarity graph; build the weighted network first

## Inputs

- Full-connectivity (unpruned) metabolite similarity network edge list with cosine similarity weights, from Alexander0/molecular_communities repository data
- GNPS/MassIVE global molecular network node list (~8.4M tandem-MS spectra) and cosine-similarity edge list

## Outputs

- communities.csv: node-to-community mapping table with one row per metabolite node and its community identifier
- Reconstructed MCN edge list (union of per-community maximum-weight spanning trees over cosine-similarity edges)
- mcn_network_statistics.csv with reproduced connectivity statistics (e.g. singleton fraction, nodes linked)

## How to apply

Load the full-connectivity metabolite similarity network — all pairwise cosine-similarity edges retained with no threshold applied — from the molecular_communities repository data (Alexander0/molecular_communities), or build the weighted graph from a GNPS/MassIVE node/edge list using networkx. Run the Louvain method (e.g. python-louvain or NetworkX community detection) on the unpruned network, letting modularity optimization determine the number and sizes of communities rather than fixing them in advance. Map each metabolite node (~8.4M in the global GNPS/MassIVE network) to its detected community identifier, writing the node-to-community mapping to communities.csv with one row per node. Optionally, within each Louvain community compute a maximum-weight spanning tree over the cosine-similarity edges and union the per-community trees to form the reconstructed MCN. Rationale: conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, whereas Louvain on the unpruned graph captures communities that arise naturally from the input data.

## Related tools

- **Louvain** (Modularity-optimization community-detection algorithm applied to the unpruned metabolite similarity network to produce the molecular communities)
- **networkx** (Builds the weighted molecular similarity graph from the GNPS/MassIVE edge list and runs Louvain community detection over its ~8.4M nodes)
- **molecular_communities** (Source repository providing the full-connectivity metabolite similarity network data used for Louvain-based MCN reconstruction) — https://github.com/Alexander0/molecular_communities

## Examples

```
python -c "import networkx as nx, community as community_louvain; G=nx.read_weighted_edgelist('edges.csv', delimiter=','); part=community_louvain.best_partition(G); import csv; w=csv.writer(open('communities.csv','w')); w.writerow(['node','community']); [w.writerow([n,c]) for n,c in part.items()]"
```

## Evaluation signals

- communities.csv exists and contains exactly one community assignment per network node
- The number of communities emerged from modularity optimization rather than being predefined
- Singleton fraction reduced from 57.4% (conventional cosine-0.7 network) to 4.9% (MCN)
- ~95% of nodes (~8M of ~8.4M) linked in the reconstructed MCN
- ~3.5 million additional nodes connected to an annotated (library-identified) node relative to the conventional network

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, so results depend on using the unpruned network
- No changelog found in the repository README, limiting traceability of code/data versions

## Evidence

- [methods] Louvain on unpruned network without predefined community counts: [evidence span withheld — anchor status: not_found]
- [results] Singleton reduction statistic: [evidence span withheld — anchor status: not_found]
