---
name: lc-ms-ms-gc-ms-network-comparison
description: Use when you have MCNs derived from multiple mass-spectrometry data types
  (e.g. LC-MS/MS and EI GC-MS) and need to assess whether network-level properties
  such as modularity or scale-free degree behaviour are consistent across platforms.
when_to_use_negative:
- When the input is a raw spectral file (mzML, .d, .mgf) rather than an already-built
  MCN graph — building the network is a separate upstream skill.
- When the goal is to compare community *membership* between platforms (e.g. shared
  metabolites) rather than network-level statistics such as modularity or degree distributions.
- When only a single data type (one LC-MS/MS run or one EI GC-MS run) is available
  and cross-platform consistency cannot be evaluated.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0092
tools:
- name: NetworkX
  role: Compute modularity and node degree distributions of each MCN graph
  repo: https://github.com/Alexander0/molecular_communities
- name: NumPy
  role: Numerical aggregation of degree counts and distribution statistics
  repo: https://github.com/Alexander0/molecular_communities
- name: Pandas
  role: Tabulate per-dataset modularity and degree statistics and write network_topology_results.csv
  repo: https://github.com/Alexander0/molecular_communities
- name: Matplotlib
  role: Plot degree distributions on log-log axes for power-law assessment (Supplemental
    Figure S18)
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:08781fcdca4f94ed3b5aab78bb77fa53d3c2b3111f9e9890b62e02784b23ea34
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Matplotlib
  - NetworkX
  - NumPy
  - Pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lc-ms-ms-gc-ms-network-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Compares the topology of molecular community networks (MCNs) built from both LC-MS/MS and EI GC-MS untargeted metabolomics data, quantifying modularity and power-law degree distributions to test whether community structure is a general property across mass-spectrometry platforms.

## When to use

Use when you have MCNs derived from multiple mass-spectrometry data types (e.g. LC-MS/MS and EI GC-MS) and need to assess whether network-level properties such as modularity or scale-free degree behaviour are consistent across platforms.

## When NOT to use

- When the input is a raw spectral file (mzML, .d, .mgf) rather than an already-built MCN graph — building the network is a separate upstream skill.
- When the goal is to compare community *membership* between platforms (e.g. shared metabolites) rather than network-level statistics such as modularity or degree distributions.
- When only a single data type (one LC-MS/MS run or one EI GC-MS run) is available and cross-platform consistency cannot be evaluated.

## Inputs

- MCN network structures for the examined LC-MS/MS datasets (from the molecular_communities repository output, as referenced in Table S1)
- MCN network structures for the examined EI GC-MS datasets (from the molecular_communities repository output, as referenced in Table S1)
- MCN-derived molecular community (cluster) assignments for each network node

## Outputs

- Per-dataset modularity values (intra-cluster vs. inter-cluster link ratio) aggregated in network_topology_results.csv
- Degree-distribution summary statistics for each network in network_topology_results.csv
- Log-log degree-distribution plots (Matplotlib) analogous to Supplemental Figure S18

## How to apply

Load each precomputed MCN for the examined LC-MS/MS and EI GC-MS datasets (as referenced in Table S1 and Supplemental Figure S18). For every network, compute modularity with NetworkX as the ratio of intra-cluster links to inter-cluster links across the MCN-derived molecular communities, and tabulate the node degree distributions using NetworkX and NumPy. Plot the degree distributions on log-log axes with Matplotlib and assess the power-law fit by the near-straight-line criterion described for Supplemental Figure S18. Aggregate per-dataset modularity values and degree-distribution summary statistics with Pandas and write them to network_topology_results.csv.

## Related tools

- **NetworkX** (Compute modularity and node degree distributions of each MCN graph) — https://github.com/Alexander0/molecular_communities
- **NumPy** (Numerical aggregation of degree counts and distribution statistics) — https://github.com/Alexander0/molecular_communities
- **Pandas** (Tabulate per-dataset modularity and degree statistics and write network_topology_results.csv) — https://github.com/Alexander0/molecular_communities
- **Matplotlib** (Plot degree distributions on log-log axes for power-law assessment (Supplemental Figure S18)) — https://github.com/Alexander0/molecular_communities

## Examples

```
import networkx as nx, numpy as np, pandas as pd, matplotlib.pyplot as plt; rows=[]; [rows.append({'dataset': d, 'modularity': nx.algorithms.community.quality.modularity(G, communities), 'degrees': np.bincount([deg for _, deg in G.degree()])}) for d, G, communities in mcn_data]; pd.DataFrame(rows).to_csv('network_topology_results.csv')
```

## Evaluation signals

- Every examined dataset (both LC-MS/MS and EI GC-MS) should show consistently high modularity, i.e. intra-cluster links dominating inter-cluster links.
- Each degree distribution should appear near-linear when plotted on log-log axes, matching the small-world power-law behaviour reported in Supplemental Figure S18.
- network_topology_results.csv must contain one row per examined dataset with modularity and degree-distribution summary columns populated.
- Modularity values computed with NetworkX should match the intra-/inter-cluster link ratio definition used in the article (Table S1 reference).
- Cross-platform comparison should show no systematic divergence in modularity between LC-MS/MS and EI GC-MS networks.

## Limitations

- Power-law assessment is qualitative (near-straight-line criterion on log-log axes) rather than a formal statistical goodness-of-fit test.
- The skill depends on MCN community assignments already being defined; it does not re-derive community structure.
- Cross-platform generalization is limited to the LC-MS/MS and EI GC-MS data types enumerated in Table S1; other acquisition modes are not covered by the source evidence.

## Evidence

- [supplementary] modularity — the ratio of intra-cluster vs. inter-cluster links — indicative of stronger community structure: [evidence span withheld — anchor status: not_found]
- [supplementary] degree distributions on log-log axes: [evidence span withheld — anchor status: not_found]
