---
name: graph-community-detection
description: Use when two ICA decompositions of different transcriptomic datasets
  yield metagene sets that must be matched to assess component reproducibility via
  RBH graph matching and community metrics.
when_to_use_negative:
- The ICA decompositions come from the same dataset/identical sample set, where components
  can be matched directly by ordering or correlation without RBH graph matching
- The goal is comparing ICA components to external annotations (e.g. functional enrichment)
  rather than cross-decomposition reproducibility of metagenes
- The data are not ICA metagene weight vectors (e.g. raw expression matrices or already-clustered
  feature tables), so metagene-level RBH matching is meaningless
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3360
tools:
- name: Python
  role: Implements the RBH computation, networkx graph construction, community detection,
    and metric computation
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Jupyter notebook
  role: Interactive environment accompanying the review in which the RBH graph matching
    and network metrics workflow is executed
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:eb242e86fac5b2c7670212497b05741030b83f6cd32797a0d50bc83f82c9aa65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - Jupyter notebook
  - Python
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# graph-community-detection

## Summary

Matches metagenes (sk gene-weight vectors) from two ICA decompositions as reciprocal best hits (RBH) and builds an undirected networkx graph whose community structure and topology quantify the reproducibility of ICA components across cancer omics datasets.

## When to use

Use when two ICA decompositions of different transcriptomic datasets yield metagene sets that must be matched to assess component reproducibility via RBH graph matching and community metrics.

## When NOT to use

- The ICA decompositions come from the same dataset/identical sample set, where components can be matched directly by ordering or correlation without RBH graph matching
- The goal is comparing ICA components to external annotations (e.g. functional enrichment) rather than cross-decomposition reproducibility of metagenes
- The data are not ICA metagene weight vectors (e.g. raw expression matrices or already-clustered feature tables), so metagene-level RBH matching is meaningless

## Inputs

- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset A (pandas DataFrame)
- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset B (pandas DataFrame)

## Outputs

- rbh_edge_list.csv — reciprocal best hit edge list of 'orthologous' metagenes
- rbh_network_metrics.csv — modularity, average clustering coefficient, and community membership of each metagene

## How to apply

Load the sk (gene-weight) vectors of the two ICA decompositions into pandas DataFrames and compute pairwise correlations between dataset A and dataset B sk vectors to obtain a metagene-by-metagene similarity matrix. For each metagene in each dataset identify its best-correlated counterpart in the other dataset and retain only reciprocal best hits (RBH) as edges of 'orthologous' metagenes. Build an undirected graph from the RBH edge list with networkx and run community detection to assign each metagene a cluster membership. Compute modularity from the community detection and the average clustering coefficient of the graph, then write the RBH edge list to rbh_edge_list.csv and the metrics table (modularity, average clustering coefficient, community structure) to rbh_network_metrics.csv. Judge correctness qualitatively by expert comparison with the metrics reported in Sections 2.4 and 3.6 of the review, since no quantitative threshold is given in the text.

## Related tools

- **Python** (Implements the RBH computation, networkx graph construction, community detection, and metric computation) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Jupyter notebook** (Interactive environment accompanying the review in which the RBH graph matching and network metrics workflow is executed) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
import pandas as pd, networkx as nx; S = dfA_sk.T.corrwith(dfB_sk.T, axis=1); G = nx.from_pandas_edgelist(rbh_pairs, source='A', target='B'); nx.algorithms.community.modularity(G, nx.algorithms.community.greedy_modularity_communities(G))
```

## Evaluation signals

- rbh_edge_list.csv exists and contains only reciprocal best-hit pairs (each edge's endpoints are each other's best-correlated counterpart)
- rbh_network_metrics.csv exists and contains columns for modularity, average clustering coefficient, and community membership
- Every metagene appearing in rbh_edge_list.csv has a non-empty community assignment in rbh_network_metrics.csv
- Average clustering coefficient is within [0, 1] and modularity is a plausible value (typically < 1) for the constructed graph
- Metrics are qualitatively consistent with those reported in Sections 2.4 and 3.6 of the review (expert review; no quantitative threshold is given)

## Limitations

- The section text provides no quantitative threshold for accepting the reproduced metrics; validation relies on qualitative comparison and expert review
- RBH matching keeps only strictly reciprocal best hits, so metagenes without a mutual best match are dropped from the graph entirely
- Results depend on the ICA decompositions themselves; BIODICA notes it is not necessary to re-compute the ICA decomposition each time and different procedures can be applied to results computed earlier or even not with BIODICA

## Evidence

- [methods] RBH graph matching workflow: [evidence span withheld — anchor status: not_found]
- [intro] Tooling from article entities: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [abstract] Notebook provenance: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [readme] Reproducibility of decompositions (BIODICA README): [evidence span withheld — anchor status: external_doc]
