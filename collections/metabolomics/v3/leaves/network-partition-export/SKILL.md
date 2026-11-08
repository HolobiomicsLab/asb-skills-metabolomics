---
name: network-partition-export
description: Use when a full-connectivity metabolite similarity network (all pairwise
  cosine edges retained, no threshold) needs to be partitioned into molecular communities
  without a predefined community count.
when_to_use_negative:
- Input network has already been pruned with an arbitrary cosine similarity threshold
  — MCN's premise is that such thresholds fracture natural metabolite families
- The number or sizes of communities is predefined by the analysis — this skill requires
  modularity optimization to determine community structure from the data
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Louvain
  role: Modularity-optimization algorithm applied to the unpruned full-connectivity
    network to detect molecular communities with no preset community count
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:29b1031bf406fcbd2f67627fc5ff8db38bbdefede4973504b18407bab841f839
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Louvain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# network-partition-export

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Runs modularity-based community detection (Louvain) on an unpruned, full-connectivity metabolite similarity network and exports the node-to-community partition as a structured communities.csv table. It lets community structure emerge from the data rather than imposing a preset number of clusters.

## When to use

Use when a full-connectivity metabolite similarity network (all pairwise cosine edges retained, no threshold) needs to be partitioned into molecular communities without a predefined community count.

## When NOT to use

- Input network has already been pruned with an arbitrary cosine similarity threshold — MCN's premise is that such thresholds fracture natural metabolite families
- The number or sizes of communities is predefined by the analysis — this skill requires modularity optimization to determine community structure from the data

## Inputs

- Full-connectivity metabolite similarity network from Alexander0/molecular_communities (all pairwise edges retained, no cosine threshold)
- Metabolite node list with pairwise similarity edges

## Outputs

- communities.csv — structured table with one row per metabolite node mapping each node to its detected community identifier

## How to apply

Load the full-connectivity metabolite similarity network from the molecular_communities repository data (Alexander0/molecular_communities), in which all pairwise edges are retained with no cosine similarity threshold applied. Run the Louvain method (e.g. via python-louvain or NetworkX community detection) on this unpruned network, letting modularity optimization determine the community structure rather than specifying a preset number or size of communities. Map each metabolite node in the network to its detected community identifier to form the partition. Write the node-to-community mapping to communities.csv as a structured table with one row per metabolite node. The rationale is that MCN avoids arbitrary similarity thresholds that can fracture natural metabolite families, so the partition must come from modularity optimization over the unpruned graph.

## Related tools

- **Louvain** (Modularity-optimization algorithm applied to the unpruned full-connectivity network to detect molecular communities with no preset community count)

## Examples

```
import networkx as nx, community_louvain; G = nx.read_weighted_edgelist('network.edges'); part = community_louvain.best_partition(G); [open('communities.csv','w').write(f'{n},{c}\n') for n,c in part.items()]
```

## Evaluation signals

- communities.csv exists and is populated
- communities.csv contains exactly one community assignment per network node
- The number of communities emerged from modularity optimization rather than being predefined
- Every metabolite node in the input network appears in the exported partition (complete node-to-community coverage)
- Running Louvain on the unpruned network (no threshold applied) reproduces communities that arise naturally from the input data

## Limitations

- Requires the unpruned, full-connectivity network; applying it to a threshold-pruned network contradicts the MCN approach of rescuing otherwise hidden metabolite connections
- Louvain partitioning gives no control over community number or sizes — results are determined solely by modularity optimization on the input graph

## Evidence

- [other] louvain_unpruned_network: [evidence span withheld — anchor status: not_found]
- [abstract] mcn_threshold_rationale: [evidence span withheld — anchor status: not_found]
