---
name: small-world-network-assessment
description: Use when you need to assess whether a metabolomics molecular network
  exhibits small-world properties — high modularity with community-dominant links
  and near-linear log-log degree distributions — across MS platforms.
when_to_use_negative:
- Input is a raw mass-spectrometry peak list or spectral similarity matrix rather
  than an already-built MCN graph object
- The analysis goal is constructing or optimizing molecular communities (molecular
  community networking) rather than evaluating the topology of an existing network
- The network is from a data type not examined in the article (e.g. non-MS omics graphs)
  where the small-world / modularity criteria may not apply
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: NetworkX
  role: Compute network modularity and node degree counts on the MCN graph structures
  repo: https://github.com/Alexander0/molecular_communities
- name: NumPy
  role: Numerically aggregate and summarize node degree counts for each network
  repo: https://github.com/Alexander0/molecular_communities
- name: Pandas
  role: Tabulate per-dataset modularity values and degree-distribution summary statistics
  repo: https://github.com/Alexander0/molecular_communities
- name: Matplotlib
  role: Plot degree distributions on log-log axes to assess power-law fit
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:506afd8ac1ca2567081920499c4782cb90eacf1242851a8fbc4762f90b0772d8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Matplotlib
  - NetworkX
  - NumPy
  - Pandas
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# small-world-network-assessment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies topological signatures of molecular community networks (MCNs) built from untargeted metabolomics data, measuring modularity (intra- vs. inter-cluster link ratio) and power-law degree distributions across LC-MS/MS and EI GC-MS datasets to confirm small-world structure.

## When to use

Use when you need to assess whether a metabolomics molecular network exhibits small-world properties — high modularity with community-dominant links and near-linear log-log degree distributions — across MS platforms.

## When NOT to use

- Input is a raw mass-spectrometry peak list or spectral similarity matrix rather than an already-built MCN graph object
- The analysis goal is constructing or optimizing molecular communities (molecular community networking) rather than evaluating the topology of an existing network
- The network is from a data type not examined in the article (e.g. non-MS omics graphs) where the small-world / modularity criteria may not apply

## Inputs

- MCN network structures (graph objects) from molecular_communities repository output for LC-MS/MS and EI GC-MS datasets
- Dataset metadata mapping networks to the datasets listed in Table S1

## Outputs

- Per-dataset modularity values (intra-cluster vs. inter-cluster link ratios)
- Degree-distribution summary statistics (log-log near-linearity assessment)
- network_topology_results.csv

## How to apply

Load the MCN network structures for each examined LC-MS/MS and EI GC-MS dataset (as referenced in Table S1 and Supplemental Figure S18) from the molecular_communities repository output. For each network, compute modularity with NetworkX as the ratio of intra-cluster links to inter-cluster links across the MCN-derived molecular communities. Separately, count node degrees with NetworkX and NumPy, then plot the degree distribution on log-log axes with Matplotlib and assess the power-law fit by the near-straight-line criterion described for Supplemental Figure S18. Accept the result only if every dataset shows intra-cluster links dominating inter-cluster links (consistently high modularity) and a degree distribution that is near-linear on log-log scale, matching the reported small-world power-law behavior. Tabulate per-dataset modularity values and degree-distribution summary statistics into network_topology_results.csv.

## Related tools

- **NetworkX** (Compute network modularity and node degree counts on the MCN graph structures) — https://github.com/Alexander0/molecular_communities
- **NumPy** (Numerically aggregate and summarize node degree counts for each network) — https://github.com/Alexander0/molecular_communities
- **Pandas** (Tabulate per-dataset modularity values and degree-distribution summary statistics) — https://github.com/Alexander0/molecular_communities
- **Matplotlib** (Plot degree distributions on log-log axes to assess power-law fit) — https://github.com/Alexander0/molecular_communities

## Examples

```
import networkx as nx; G = nx.read_gpickle('mcn_lcmsms.pkl'); m = nx.algorithms.community.modularity(G, communities); deg = np.bincount([d for _, d in G.degree()])
```

## Evaluation signals

- Every examined dataset (LC-MS/MS and EI GC-MS, per Table S1) shows consistently high modularity — intra-cluster links dominating inter-cluster links
- Each network's degree distribution is near-linear on log-log axes, consistent with a power-law small-world topology
- Per-dataset modularity values and degree-distribution summary statistics are fully tabulated in network_topology_results.csv without missing entries
- Modularity computed per dataset matches the ratio-of-intra-to-inter-cluster-links definition, not an alternative modularity formulation

## Limitations

- Modularity is defined here specifically as the ratio of intra-cluster vs. inter-cluster links across MCN-derived molecular communities, so results are not directly comparable to standard modularity (e.g. Newman) implementations
- Power-law assessment relies on a near-straight-line criterion on log-log degree plots rather than a formal statistical fit
- The skill was validated only on the LC-MS/MS and EI GC-MS datasets examined in the article (Table S1); generalization to other data types is untested

## Evidence

- [supplementary] For all examined data, both LC-MS/MS and EI GC-MS (Table S1), the MCNs consistently exhibit high modularity — the ratio of intra-cluster vs. inter-cluster links: [evidence span withheld — anchor status: not_found]
- [supplementary] indicative of stronger community structure: [evidence span withheld — anchor status: not_found]
- [supplementary] Matching small-world power-law behavior: [evidence span withheld — anchor status: not_found]
