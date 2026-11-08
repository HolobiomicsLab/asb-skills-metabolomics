---
name: graph-modularity-optimization
description: Use when a full-connectivity metabolite similarity network (all pairwise
  edges retained, no cosine threshold) must be partitioned into communities without
  a predefined cluster count, as in MCN's community-detection step.
when_to_use_negative:
- The network has already been pruned with an arbitrary cosine similarity threshold
  — conventional thresholded molecular networking is exactly the approach MCN was
  designed to avoid, since arbitrary thresholds can fracture natural metabolite families
- A predefined number or size of communities must be enforced — this skill relies
  on modularity optimization without preset community counts
edam_operation: http://edamontology.org/operation_3960
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: Louvain
  role: Modularity-optimization community-detection algorithm run on the unpruned
    metabolite similarity network to produce the molecular community partition
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:adf6f05c2da704c17e23e5b47bbdf695ab202bec90e33c6c69f349b0b542e5fb
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Louvain
  edam_operation: http://edamontology.org/operation_3960
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# graph-modularity-optimization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Apply the Louvain modularity-optimization algorithm to partition an unpruned, full-connectivity metabolite similarity network into molecular communities, letting community structure emerge naturally from the data rather than fixing a preset number or size of communities.

## When to use

Use when a full-connectivity metabolite similarity network (all pairwise edges retained, no cosine threshold) must be partitioned into communities without a predefined cluster count, as in MCN's community-detection step.

## When NOT to use

- The network has already been pruned with an arbitrary cosine similarity threshold — conventional thresholded molecular networking is exactly the approach MCN was designed to avoid, since arbitrary thresholds can fracture natural metabolite families
- A predefined number or size of communities must be enforced — this skill relies on modularity optimization without preset community counts

## Inputs

- Unpruned full-connectivity metabolite similarity network (all pairwise edges retained, no cosine similarity threshold)

## Outputs

- communities.csv — structured node-to-community mapping with one row per metabolite node and one community assignment per network node

## How to apply

Load the full-connectivity metabolite similarity network from the molecular_communities repository data, in which all pairwise edges are retained with no cosine similarity threshold applied. Run the Louvain method (e.g. python-louvain / NetworkX community detection) on this unpruned network so that modularity optimization alone determines the community structure — do not specify a preset number or sizes of communities. Map each metabolite node to its detected community identifier to form the partition, and write the node-to-community mapping to communities.csv with one row per metabolite node. The rationale is that MCN optimizes connectivity for each metabolite, rescuing relationships that arbitrary similarity thresholds would fracture.

## Related tools

- **Louvain** (Modularity-optimization community-detection algorithm run on the unpruned metabolite similarity network to produce the molecular community partition) — https://github.com/Alexander0/molecular_communities

## Examples

```
python -m community_detection --input full_similarity_network --algorithm louvain --output communities.csv
```

## Evaluation signals

- communities.csv exists and contains exactly one community assignment per network node
- Every metabolite node in the network is present in the output mapping (row count equals node count)
- The number of communities emerged from modularity optimization rather than being predefined
- All pairwise edges of the input network were retained (no cosine similarity threshold was applied before clustering)

## Limitations

- Conventional molecular networking uses arbitrary similarity thresholds that can fracture natural metabolite families; applying such thresholds before this skill defeats its purpose
- Community structure is determined solely by modularity optimization on the input network — no external constraint on the number or sizes of communities is used

## Evidence

- [methods] louvain_unpruned: [evidence span withheld — anchor status: not_found]
- [abstract] threshold_critique: "using arbitrary similarity thresholds that can fracture natural metabolite families"
- [abstract] mcn_rationale: "We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections."
