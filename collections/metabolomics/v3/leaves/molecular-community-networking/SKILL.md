---
name: molecular-community-networking
description: Use when analyzing untargeted metabolomics MS/MS spectra where cosine-threshold
  molecular networks fracture metabolite families, and you want community structure
  (e.g. community assignments for every node) to emerge from modularity optimization
  on the unpruned similarity network.
when_to_use_negative:
- When a predefined number or size of metabolite communities is required — MCN specifically
  lets the Louvain modularity optimization determine community structure rather than
  fixing it.
- When the goal is a conventional cosine-threshold molecular network (e.g. cosine
  similarity > 0.7 edges only) — pruning is exactly what MCN avoids, as it can fracture
  natural metabolite families.
- When the input is already a pruned/thresholded molecular network with most edges
  removed — running Louvain on the pruned graph would not recover the full-connectivity
  structure MCN relies on.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Louvain
  role: Community-detection algorithm run on the unpruned metabolite similarity network
    to derive molecular communities via modularity optimization
  repo: https://github.com/Alexander0/molecular_communities
- name: molecular_communities (Alexander0/molecular_communities)
  role: Reference code and data repository for building the conventional cosine-similarity
    molecular network, running MCN, and inspecting recovered communities
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:244f3ce4cbcc7d22db5081cf75d27be7a26ba0b1222d3a0ad224aebbac0c258c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Louvain
  - molecular_communities (Alexander0/molecular_communities)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# molecular-community-networking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Molecular community networking (MCN) partitions untargeted metabolomics MS/MS similarity networks by optimizing connectivity for each metabolite via modularity-based community detection (Louvain) on the full-connectivity network, instead of pruning edges with an arbitrary cosine-similarity threshold. It rescues 'hidden' metabolite relationships that conventional molecular networking loses, as demonstrated by the discovery of novel dipeptide-conjugated bile acids linked to Bifidobacterium.

## When to use

Use when analyzing untargeted metabolomics MS/MS spectra where cosine-threshold molecular networks fracture metabolite families, and you want community structure (e.g. community assignments for every node) to emerge from modularity optimization on the unpruned similarity network.

## When NOT to use

- When a predefined number or size of metabolite communities is required — MCN specifically lets the Louvain modularity optimization determine community structure rather than fixing it.
- When the goal is a conventional cosine-threshold molecular network (e.g. cosine similarity > 0.7 edges only) — pruning is exactly what MCN avoids, as it can fracture natural metabolite families.
- When the input is already a pruned/thresholded molecular network with most edges removed — running Louvain on the pruned graph would not recover the full-connectivity structure MCN relies on.

## Inputs

- Pairwise MS/MS cosine-similarity matrix or similarity network over untargeted metabolomics spectra (all pairwise edges retained, unpruned)
- Metabolite feature table / spectral IDs from the untargeted metabolomics dataset (e.g. bile acid dataset, Figures 2 and S15-16)
- Optional supplemental metadata mapping detected molecules to biological/microbial associations

## Outputs

- Node-to-community mapping table (e.g. communities.csv) with one row per metabolite node and its Louvain community identifier
- Identified 'hidden' or rescued metabolite connections (e.g. dipeptide-conjugated bile acids) not present in threshold-pruned networks
- Annotated associations of recovered molecules with microbial species (e.g. Bifidobacterium breve) when such metadata is available

## How to apply

Build the pairwise MS/MS cosine-similarity network over the metabolite spectra but retain all pairwise edges (no similarity-threshold pruning) — MCN's core rationale is that arbitrary thresholds 'can fracture natural metabolite families'. Run the Louvain method on this unpruned network without specifying a predefined number or size of communities, letting modularity optimization determine the partition. Map each metabolite node to its detected community identifier and export the node-to-community mapping (e.g. communities.csv), one row per metabolite. To surface 'hidden' connections, inspect the recovered communities for molecules (e.g. dipeptide-conjugated bile acids) that a conventional threshold-based network would have dropped; optionally annotate such molecules with microbial associations from supplemental materials (e.g. Bifidobacterium breve in infant stool). Use the Alexander0/molecular_communities repository code and data as the reference implementation.

## Related tools

- **Louvain** (Community-detection algorithm run on the unpruned metabolite similarity network to derive molecular communities via modularity optimization) — https://github.com/Alexander0/molecular_communities
- **molecular_communities (Alexander0/molecular_communities)** (Reference code and data repository for building the conventional cosine-similarity molecular network, running MCN, and inspecting recovered communities) — https://github.com/Alexander0/molecular_communities

## Examples

```
from community import best_partition; import networkx as nx; G = nx.read_graphml('mcn_full_connectivity.graphml'); partition = best_partition(G)
```

## Evaluation signals

- communities.csv exists and contains exactly one community assignment per network node (row count equals node count of the unpruned network).
- The number of communities is an emergent output of Louvain modularity optimization, not a value that was predefined or hardcoded.
- All pairwise edges of the metabolite similarity network were retained as input — verify no cosine-similarity threshold was applied before community detection.
- Recovered communities contain metabolite connections (e.g. dipeptide-conjugated bile acids) that are absent from a conventional threshold-based molecular network on the same spectra.
- For the bile acid demonstration, MCN reproduces the association of a novel dipeptide-conjugated bile acid with Bifidobacterium breve as reported in the Supplemental materials.

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families — hence MCN avoids them, but users should be aware MCN outputs depend on the full-connectivity network construction.
- Details of the Bifidobacterium breve association and the recovered bile acid identifications reside in the Supplemental materials rather than the main text; validation requires accessing those SI materials.
- No changelog was found for the molecular_communities repository, so version drift in the reference implementation is undocumented.

## Evidence

- [other] Louvain on unpruned network without predefined communities: [evidence span withheld — anchor status: not_found]
- [other] Workflow step: run Louvain on full-connectivity network: [evidence span withheld — anchor status: not_found]
- [other] Validation of communities.csv: [evidence span withheld — anchor status: not_found]
- [supplementary] B. breve association: [evidence span withheld — anchor status: not_found]
