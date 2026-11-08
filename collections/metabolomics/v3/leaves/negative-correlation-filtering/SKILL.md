---
name: negative-correlation-filtering
description: Use when a correlation network built from abundance data carries both
  positive and negative edges, and the analysis goal (e.g., sample-specific subnetworks
  or topological comparison) requires retaining only co-occurrence (positive) edges.
when_to_use_negative:
- The metric matrix is non-negative by construction (e.g., similarity or Jaccard),
  so there are no negative edges to remove — filtering is a no-op.
- Negative (mutual-exclusion) edges are themselves the biological object of study
  and must be preserved in the topology.
- The goal is a full multi-omics cross-layer network where inhibitory cross-layer
  relationships are of interest.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaNet
  role: Provides extract_sample_net() with remove_negative=TRUE to strip negative
    edges when building sample-specific subnetworks, and net_par(remove_negative=TRUE)
    for consistent topology computation.
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: Core graph engine underlying MetaNet's network construction and topological
    index calculation.
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: Execution environment for the MetaNet filtering and topological-analysis workflow.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:7191ff039ad6470999c6830caccf4050c114211ae12657b31c752ac9e753e960
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# negative-correlation-filtering

## Summary

Filters signed correlation-based co-occurrence networks to retain only positive edges (remove_negative=TRUE) before subnetwork extraction and topological computation, so downstream indices are not distorted by indirect/antagonistic associations.

## When to use

Use when a correlation network built from abundance data carries both positive and negative edges, and the analysis goal (e.g., sample-specific subnetworks or topological comparison) requires retaining only co-occurrence (positive) edges.

## When NOT to use

- The metric matrix is non-negative by construction (e.g., similarity or Jaccard), so there are no negative edges to remove — filtering is a no-op.
- Negative (mutual-exclusion) edges are themselves the biological object of study and must be preserved in the topology.
- The goal is a full multi-omics cross-layer network where inhibitory cross-layer relationships are of interest.

## Inputs

- Feature abundance table (e.g., bacterial taxa-by-sample abundance matrix, 'micro')
- Baseline co-occurrence network object (co_net) with signed correlation edges
- Sample/timepoint metadata (e.g., timepoints A–D) defining which samples define each subnetwork

## Outputs

- Positive-edge-only sample-specific subnetwork objects (one per timepoint/group)
- Topological index table (sample_net_topology.csv) with modularity, average path length, global efficiency, clustering coefficient, natural connectivity per timepoint

## How to apply

Build the baseline co-occurrence network in R with MetaNet (c_net_calculate + c_net_build with a correlation threshold such as r_threshold = 0.6–0.65), then drop negative edges either at build time or during subnetwork extraction. For sample-specific subnetworks, call extract_sample_net() with remove_negative=TRUE and the fast option so only positive edges present in each sample's abundance profile are retained; apply remove_negative=TRUE again in net_par() when computing topology so the indices match the filtered networks. The rationale is that negative correlations reflect mutual exclusion or indirect effects rather than true co-occurrence, and the article itself warns that correlation-threshold filtering choices can bias network topology and downstream biological interpretation. Finally compute topological parameters (modularity, average path length, global efficiency, clustering coefficient, natural connectivity) on the positive-only networks and export them (e.g., sample_net_topology.csv) for comparison against a baseline timepoint.

## Related tools

- **MetaNet** (Provides extract_sample_net() with remove_negative=TRUE to strip negative edges when building sample-specific subnetworks, and net_par(remove_negative=TRUE) for consistent topology computation.) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (Core graph engine underlying MetaNet's network construction and topological index calculation.) — https://github.com/Asa12138/MetaNet
- **R** (Execution environment for the MetaNet filtering and topological-analysis workflow.)

## Examples

```
lib(MetaNet); net <- c_net_build(c_net_calculate(t(micro)), r_threshold = 0.6); sub <- extract_sample_net(net, micro, remove_negative = TRUE, fast = TRUE)
```

## Evaluation signals

- After extraction, every edge in each timepoint subnetwork corresponds to a positive correlation present in that sample's abundance profile (no negative edges remain).
- Topological parameters computed with net_par(remove_negative=TRUE) match indices computed on the filtered subnetworks directly.
- sample_net_topology.csv has exactly one row per timepoint (A–D) and one column per topological index.
- Validation: each exposure timepoint B–D shows higher modularity and average path length, and lower global efficiency, clustering coefficient, and natural connectivity, than baseline A.

## Limitations

- Threshold selection for correlation filtering is subjective and can bias network topology and downstream biological interpretation.
- Correlation-based networks may overlook complex, non-linear, or indirect associations, so positive-edge filtering is a simplification of interaction biology.
- Removing negative edges before subnetwork extraction can reduce network density, which affects modularity, efficiency, and natural-connectivity values.

## Evidence

- [methods] Sample-specific subnetwork extraction with negative-edge removal: [evidence span withheld — anchor status: not_found]
- [intro] Filtering caveat from article intro: [evidence span withheld — anchor status: not_found]
