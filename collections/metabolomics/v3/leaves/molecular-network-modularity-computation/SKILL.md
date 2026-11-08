---
name: molecular-network-modularity-computation
description: Use when you have MCN network structures (from molecular_communities
  repository outputs) for LC-MS/MS or EI GC-MS datasets and need to quantify community
  structure via modularity and power-law degree behavior.
when_to_use_negative:
- You need to construct or cluster the molecular network itself (e.g., define similarity
  thresholds or MCN communities) rather than measure topology of already-derived MCNs.
- Your input is raw mass-spectrometry peak data (mzML/feature tables) rather than
  built network graph objects.
- You are evaluating molecular networking methods that use arbitrary similarity thresholds
  — this skill characterizes topology, it does not compare networking algorithms.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: NetworkX
  role: Compute modularity and node degrees of MCN graphs
  repo: https://github.com/Alexander0/molecular_communities
- name: NumPy
  role: Numerical handling of degree counts for degree-distribution statistics
  repo: https://github.com/Alexander0/molecular_communities
- name: Pandas
  role: Tabulate per-dataset modularity and degree statistics into network_topology_results.csv
  repo: https://github.com/Alexander0/molecular_communities
- name: Matplotlib
  role: Plot degree distributions on log-log axes for power-law assessment
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:a59f7a5bf212f1c932d01296afe8448824cfaa7dc9dd2ebe52125af04a6586a4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Matplotlib
  - NetworkX
  - NumPy
  - Pandas
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-network-modularity-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes modularity (the ratio of intra-cluster to inter-cluster links) and log-log degree distributions for molecular community networks (MCNs) derived from LC-MS/MS and EI GC-MS metabolomics data, to quantify whether metabolites group into communities with small-world power-law structure.

## When to use

Use when you have MCN network structures (from molecular_communities repository outputs) for LC-MS/MS or EI GC-MS datasets and need to quantify community structure via modularity and power-law degree behavior.

## When NOT to use

- You need to construct or cluster the molecular network itself (e.g., define similarity thresholds or MCN communities) rather than measure topology of already-derived MCNs.
- Your input is raw mass-spectrometry peak data (mzML/feature tables) rather than built network graph objects.
- You are evaluating molecular networking methods that use arbitrary similarity thresholds — this skill characterizes topology, it does not compare networking algorithms.

## Inputs

- MCN network structures (graphs) for examined LC-MS/MS and EI GC-MS datasets from the molecular_communities repository output
- MCN-derived molecular community (cluster) assignments per network

## Outputs

- Per-dataset modularity values (intra-cluster vs. inter-cluster link ratios)
- Degree-distribution summary statistics per network
- network_topology_results.csv (tabulated modularity and degree-distribution statistics)
- Log-log degree distribution plots (as in Supplemental Figure S18)

## How to apply

Load the MCN network structures for each examined dataset (as referenced in Table S1 and Supplemental Figure S18) from the molecular_communities repository output. For each network, compute modularity with NetworkX as the ratio of intra-cluster links to inter-cluster links across the MCN-derived molecular communities. Compute node degrees with NetworkX and NumPy, then plot degree distributions on log-log axes with Matplotlib and assess the power-law fit by the near-straight-line criterion described for Supplemental Figure S18. Tabulate per-dataset modularity values and degree-distribution summary statistics (e.g., via Pandas) and write them to network_topology_results.csv.

## Related tools

- **NetworkX** (Compute modularity and node degrees of MCN graphs) — https://github.com/Alexander0/molecular_communities
- **NumPy** (Numerical handling of degree counts for degree-distribution statistics) — https://github.com/Alexander0/molecular_communities
- **Pandas** (Tabulate per-dataset modularity and degree statistics into network_topology_results.csv) — https://github.com/Alexander0/molecular_communities
- **Matplotlib** (Plot degree distributions on log-log axes for power-law assessment) — https://github.com/Alexander0/molecular_communities

## Examples

```
import networkx as nx; m = nx.community.modularity(G, communities); degs = np.array([d for _, d in G.degree()]); pd.DataFrame({'dataset': name, 'modularity': [m], 'mean_degree': [degs.mean()]}).to_csv('network_topology_results.csv', mode='a')
```

## Evaluation signals

- Every examined dataset (both LC-MS/MS and EI GC-MS, per Table S1) shows consistently high modularity, with intra-cluster links dominating inter-cluster links.
- Degree distributions are near-linear on log-log axes, matching the reported small-world power-law behavior (Supplemental Figure S18 criterion).
- network_topology_results.csv contains one row per examined dataset with modularity and degree-distribution summary statistics.
- Modularity values are reproducible when recomputed from the repository's MCN network structures.

## Limitations

- The modularity power-law assessment relies on the near-straight-line criterion on log-log plots, which is a visual rather than formal statistical test.
- Conclusions are grounded only in the datasets examined in Table S1 (LC-MS/MS and EI GC-MS); generalization to other data types is not established by this skill.
- The repository README provides no additional documentation (no changelog found), so implementation details must be inferred from the article workflow.

## Evidence

- [results] For all examined data, both LC-MS/MS and EI GC-MS (Table S1), the MCNs consistently exhibit high modularity: [evidence span withheld — anchor status: not_found]
- [supplementary] Near-linear log-log degree distributions indicate power-law/small-world structure: [evidence span withheld — anchor status: not_found]
- [abstract] MCN community-structure claim in abstract: [evidence span withheld — anchor status: not_found]
- [other] Repository as the data source for MCN networks: [evidence span withheld — anchor status: not_found]
