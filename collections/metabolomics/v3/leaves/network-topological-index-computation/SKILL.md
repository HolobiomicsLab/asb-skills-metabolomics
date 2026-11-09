---
name: network-topological-index-computation
description: Use when per-condition or per-sample subnetworks have been extracted
  from a co-occurrence network and you need global topological indices compared against
  a baseline to detect fragmentation or restructuring under exposure or other treatments.
when_to_use_negative:
- The input is still a raw abundance table with no constructed network — build the
  co-occurrence network first (c_net_calculate/c_net_build) before computing topology.
- The goal is module/community detection or visualization of a single network rather
  than global index computation and cross-network comparison.
- The question concerns differential edges between two specific networks (intersection/union/difference)
  rather than per-sample topological index trajectories.
edam_operation: null
tools:
- name: MetaNet
  role: Provides extract_sample_net(), net_par() (global topological indices with
    remove_negative=TRUE), and c_net_compare() for baseline-vs-condition index comparison
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: Graph engine underlying MetaNet's topological calculation module
- name: R
  role: Execution environment for the MetaNet-based topology workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:76731fc4ea4c30aa7d1aca7de5bb5164bbcf9ae329ed405ccca9ca357eede549
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# network-topological-index-computation

## Summary

Computes global topological indices (modularity, average path length, global efficiency, clustering coefficient, natural connectivity) on omics co-occurrence networks and sample-specific subnetworks using MetaNet, to quantify shifts such as community fragmentation across experimental conditions.

## When to use

Use when per-condition or per-sample subnetworks have been extracted from a co-occurrence network and you need global topological indices compared against a baseline to detect fragmentation or restructuring under exposure or other treatments.

## When NOT to use

- The input is still a raw abundance table with no constructed network — build the co-occurrence network first (c_net_calculate/c_net_build) before computing topology.
- The goal is module/community detection or visualization of a single network rather than global index computation and cross-network comparison.
- The question concerns differential edges between two specific networks (intersection/union/difference) rather than per-sample topological index trajectories.

## Inputs

- Bacterial abundance table (micro) with samples grouped by condition/timepoint (A–D)
- Baseline co-occurrence network (co_net) constructed with MetaNet
- Condition/timepoint metadata mapping samples to groups

## Outputs

- Per-timepoint sample-specific subnetworks (positive-edge-only)
- sample_net_topology.csv with one row per timepoint and one column per topological index (modularity, average path length, global efficiency, clustering coefficient, natural connectivity)
- c_net_compare comparison of B–D subnetworks against the baseline A subnetwork

## How to apply

From a baseline co-occurrence network (co_net) built in MetaNet, extract each sample-specific subnetwork with extract_sample_net() using remove_negative=TRUE and the fast option, so only positive edges present in that sample's abundance profile are retained. Compute topological parameters for each subnetwork with net_par(), using the mode argument to select global indices and remove_negative=TRUE for consistency with the extracted networks; the target indices are modularity, average path length, global efficiency, clustering coefficient, and natural connectivity. Compare condition subnetworks against the baseline subnetwork with c_net_compare() to record the direction of change for each index. Assemble the per-timepoint indices into a single table (sample_net_topology.csv) with one row per timepoint and one column per index. In the source task this revealed increased modularity and average path length with decreased global efficiency, clustering coefficient, and natural connectivity at exposure timepoints B–D relative to baseline A, indicating community fragmentation.

## Related tools

- **MetaNet** (Provides extract_sample_net(), net_par() (global topological indices with remove_negative=TRUE), and c_net_compare() for baseline-vs-condition index comparison) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (Graph engine underlying MetaNet's topological calculation module)
- **R** (Execution environment for the MetaNet-based topology workflow)

## Examples

```
cor <- c_net_calculate(totu); net <- c_net_build(cor, r_threshold = 0.65)
```

## Evaluation signals

- sample_net_topology.csv has exactly one row per timepoint (A–D) and one column per topological index, with finite numeric values
- Each timepoint B–D shows higher modularity and average path length than baseline A, and lower global efficiency, clustering coefficient, and natural connectivity than baseline A
- remove_negative=TRUE was applied consistently in both extract_sample_net() and net_par(), so indices reflect positive-edge-only subnetworks
- Indices agree in direction with the c_net_compare() output for B–D vs A
- Subnetwork extraction retains only edges supported by each sample's abundance profile (no edges to features absent from that sample)

## Limitations

- Correlation-based network optimization may overlook complex, non-linear, or indirect associations
- Lack of standardized protocols for network construction (similarity metrics, thresholding, and null model selection) can bias topology and downstream interpretation
- Indices depend on the baseline co-occurrence network and threshold choices; negative correlations are discarded by the remove_negative=TRUE workflow, which can alter topology

## Evidence

- [readme] Multi-network comparison framework supporting baseline-vs-condition index comparison: [evidence span withheld — anchor status: external_doc]
