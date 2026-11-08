---
name: longitudinal-network-comparison
description: Use when a longitudinal abundance table and a pre-built co-occurrence
  network are available and the goal is to test whether per-sample subnetwork topology
  at later timepoints diverges from a baseline timepoint.
when_to_use_negative:
- The question is cross-layer (e.g. exposome–transcriptome integration) rather than
  temporal comparison of subnetworks from one omics layer — use multi-omics integration
  workflows instead
- Only a single network exists with no time-resolved abundance profiles, so no per-sample
  subnetworks can be extracted
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: Provides extract_sample_net() for per-timepoint subnetwork extraction, net_par()
    for topological indices, and c_net_compare() for baseline-vs-exposure comparison
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: Core dependency on which MetaNet's network objects and topological calculations
    are built
- name: R
  role: Execution environment for the longitudinal comparison workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:2a0cb808aaf605323004bed93ab8e16891e666e42ae14b8d98c25b19c6ae7383
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# longitudinal-network-comparison

## Summary

Compares sample-specific subnetworks extracted from a baseline co-occurrence network across ordered timepoints (A–D) using MetaNet's extract_sample_net(), net_par(), and c_net_compare() to quantify topology shifts (modularity, path length, efficiency, clustering, natural connectivity) under exposure.

## When to use

Use when a longitudinal abundance table and a pre-built co-occurrence network are available and the goal is to test whether per-sample subnetwork topology at later timepoints diverges from a baseline timepoint.

## When NOT to use

- The question is cross-layer (e.g. exposome–transcriptome integration) rather than temporal comparison of subnetworks from one omics layer — use multi-omics integration workflows instead
- Only a single network exists with no time-resolved abundance profiles, so no per-sample subnetworks can be extracted

## Inputs

- Bacterial abundance table (micro) with rows as taxa and columns as samples spanning timepoints A–D
- Baseline co-occurrence network object (co_net) built in MetaNet/igraph
- Timepoint/metadata labels assigning each sample to timepoint A, B, C, or D

## Outputs

- sample_net_topology.csv with one row per timepoint and one column per topological index (modularity, average path length, global efficiency, clustering coefficient, natural connectivity)
- c_net_compare() comparison of B–D subnetworks against baseline A

## How to apply

Load the bacterial abundance table (micro) and the baseline co-occurrence network (co_net) in R with the MetaNet package. For each timepoint (A–D), call extract_sample_net() with remove_negative=TRUE and fast=TRUE so only positive edges supported by each sample's abundance profile are retained, then compute global topological indices with net_par() (mode selecting global indices, remove_negative=TRUE for consistency). Compare the B–D subnetworks against the baseline A subnetwork with c_net_compare(), recording modularity, average path length, global efficiency, clustering coefficient, and natural connectivity. Assemble per-timepoint indices into sample_net_topology.csv with one row per timepoint and one column per index; the validation pattern is that exposure timepoints B–D show higher modularity and average path length but lower efficiency, clustering, and natural connectivity than A, indicating community fragmentation.

## Related tools

- **MetaNet** (Provides extract_sample_net() for per-timepoint subnetwork extraction, net_par() for topological indices, and c_net_compare() for baseline-vs-exposure comparison) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (Core dependency on which MetaNet's network objects and topological calculations are built)
- **R** (Execution environment for the longitudinal comparison workflow)

## Examples

```
library(MetaNet); net_B <- extract_sample_net(co_net, micro[, tp == "B"], remove_negative = TRUE, fast = TRUE); net_par(net_B, remove_negative = TRUE)
```

## Evaluation signals

- Each timepoint B–D exhibits higher modularity and average path length, and lower global efficiency, clustering coefficient, and natural connectivity, than baseline A in sample_net_topology.csv
- extract_sample_net() was run with remove_negative=TRUE and fast so extracted subnetworks contain only positive edges present in each sample's abundance profile
- net_par() used mode selecting global indices and remove_negative=TRUE, consistent with the extracted networks
- sample_net_topology.csv has exactly one row per timepoint (A–D) and one column per topological index
- All samples in the abundance table successfully map to a timepoint label before extraction

## Limitations

- Correlation-based network optimization may overlook complex, non-linear, or indirect associations
- Lack of standardized protocols for network construction (similarity metrics, thresholding, null model selection) can bias topological comparisons
- Only positive edges are retained (remove_negative=TRUE), so negative co-occurrence relationships are excluded from the topology comparison

## Evidence

- [other] Extract the sample-specific subnetwork for each timepoint (A–D) with extract_sample_net(), using the remove_negative=TRUE and fast options: [evidence span withheld — anchor status: not_found]
- [other] Compare the B–D subnetworks against the baseline A subnetwork with c_net_compare(), recording modularity, average path length, global efficiency, clustering coefficient, and natural connectivity.: [evidence span withheld — anchor status: not_found]
- [readme] MetaNet enables such comparisons by computing intersections, unions, and differences between networks (Figure 2H), offering a flexible framework for comparative and evolutionary network analysis.: [evidence span withheld — anchor status: external_doc]
- [results] reflecting community fragmentation and impaired bacterial interactions under exposure.: [evidence span withheld — anchor status: not_found]
