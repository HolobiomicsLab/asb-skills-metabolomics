---
name: cross-dataset-metagene-network-construction
description: Use when two ICA decompositions of different cancer transcriptomic datasets
  must be compared component-wise to assess reproducibility, and no one-to-one component
  ordering exists between them.
when_to_use_negative:
- The decompositions share an established component ordering, so direct per-component
  comparison suffices and RBH matching is unnecessary.
- The task is assessing ICA stability within a single dataset (e.g. via stability
  analysis in BIODICA), not reproducibility across two independent datasets.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Python
  role: Language used (3.7.3) to compute cross-dataset metagene correlations, RBH
    matching, and network metrics in the accompanying notebook
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Jupyter notebook
  role: Interactive notebook accompanying the review that illustrates the ICA workflow
    including metagene matching
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:42db8be08b7cea721b0bafa0caf873e65e1ef753d6caba352f5b849dfcd1273c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-dataset-metagene-network-construction

## Summary

Builds an RBH (reciprocal best hit) network linking metagenes (sk gene-weight vectors) from two ICA decompositions of different transcriptomic datasets, then quantifies reproducibility of components via network metrics such as modularity and average clustering coefficient.

## When to use

Use when two ICA decompositions of different cancer transcriptomic datasets must be compared component-wise to assess reproducibility, and no one-to-one component ordering exists between them.

## When NOT to use

- The decompositions share an established component ordering, so direct per-component comparison suffices and RBH matching is unnecessary.
- The task is assessing ICA stability within a single dataset (e.g. via stability analysis in BIODICA), not reproducibility across two independent datasets.

## Inputs

- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset A (pandas DataFrame)
- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset B (pandas DataFrame)

## Outputs

- rbh_edge_list.csv (RBH edge list of matched metagenes)
- rbh_network_metrics.csv (modularity, average clustering coefficient, community membership of each metagene)

## How to apply

Load the sk (gene-weight) vectors of both ICA decompositions into pandas DataFrames and compute the pairwise correlation matrix between dataset A and dataset B metagenes. For each metagene, find its best-correlated counterpart in the other dataset and keep only reciprocal best hits as edges of 'orthologous' metagenes. Build an undirected graph from the RBH edge list with networkx, then compute modularity from community detection, the average clustering coefficient, and each metagene's community membership. Write rbh_edge_list.csv and rbh_network_metrics.csv. Because the section text gives no quantitative threshold, validate by expert qualitative comparison with the metrics reported in Sections 2.4 and 3.6.

## Related tools

- **Python** (Language used (3.7.3) to compute cross-dataset metagene correlations, RBH matching, and network metrics in the accompanying notebook) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Jupyter notebook** (Interactive notebook accompanying the review that illustrates the ICA workflow including metagene matching) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
G = nx.from_pandas_edgelist(rbh_edges, source='A', target='B'); nx.community.modularity(G, nx.community.greedy_modularity_communities(G)); nx.average_clustering(G)
```

## Evaluation signals

- rbh_edge_list.csv exists and contains only reciprocal best-hit pairs (each edge (a, b) implies a's best hit in B is b AND b's best hit in A is a)
- rbh_network_metrics.csv contains modularity, average clustering coefficient, and community columns
- Community structure is non-degenerate: identified communities have interpretable typical sizes and the graph is connected or has few components
- Metrics qualitatively reproduce those reported in Sections 2.4 and 3.6 (expert review; no quantitative threshold given in the text)

## Limitations

- No quantitative validation threshold is provided in the section text; acceptance relies on qualitative comparison by expert review
- RBH matching is one-to-one, so decompositions with differing numbers of components will leave unmatched metagenes
- Results depend on the chosen correlation metric and on the stability of each ICA decomposition (BIODICA's stability analysis addresses the latter)

## Evidence

- [methods] RBH workflow step 2-4: [evidence span withheld — anchor status: not_found]
- [methods] RBH edge definition: [evidence span withheld — anchor status: not_found]
- [methods] Network metrics: [evidence span withheld — anchor status: not_found]
- [methods] Output files and validation: [evidence span withheld — anchor status: not_found]
- [results] Reproducibility rationale: [evidence span withheld — anchor status: not_found]
- [readme] Tooling evidence from README: "Jupyter Notebook containing example of Independent Component Analysis (ICA) accompanying the review in Internal Journal of Molecular Sciences (IJMS)"
