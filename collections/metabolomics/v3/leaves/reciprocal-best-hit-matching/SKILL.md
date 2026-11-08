---
name: reciprocal-best-hit-matching
description: Use when two ICA decompositions (e.g., of two cancer omics transcriptomic
  datasets) each yield metagene sk vectors and you need to assess component reproducibility
  by matching metagenes across the two decompositions.
when_to_use_negative:
- The two decompositions are not independent ICA runs on different datasets but components
  already matched by construction (e.g., ordered components from a stabilized-ica
  output)
- The goal is correlating sample mixing-coefficient vectors rather than gene-weight
  (sk) metagene vectors
- You need a quantitative reproducibility score against a fixed threshold — the article
  accepts reproduced metrics by qualitative comparison with Sections 2.4 and 3.6 (expert
  review), with no numeric cutoff
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implementation language for the RBH matching, graph construction, and metric
    computation (notebook runs on Python 3.7.3)
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Jupyter notebook
  role: Interactive notebook accompanying the review that illustrates the RBH metagene-matching
    workflow
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: pandas
  role: Loads the sk gene-weight vectors of both ICA decompositions into DataFrames
    for pairwise correlation
- name: networkx
  role: Builds the undirected RBH graph from the edge list and computes modularity,
    average clustering coefficient, and community membership
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:7f2b23d1c88b97cea2dc65489ff053a91b640b44269c10486728aba4bdf5ca99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - Jupyter notebook
  - Python
  - networkx
  - pandas
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Reciprocal best hit matching of ICA metagenes

## Summary

Matches metagenes (sk gene-weight vectors) from two ICA decompositions of different transcriptomic datasets by identifying reciprocal best-correlation pairs, then builds an RBH graph whose modularity, average clustering coefficient, and community structure quantify reproducibility of independent components.

## When to use

Use when two ICA decompositions (e.g., of two cancer omics transcriptomic datasets) each yield metagene sk vectors and you need to assess component reproducibility by matching metagenes across the two decompositions.

## When NOT to use

- The two decompositions are not independent ICA runs on different datasets but components already matched by construction (e.g., ordered components from a stabilized-ica output)
- The goal is correlating sample mixing-coefficient vectors rather than gene-weight (sk) metagene vectors
- You need a quantitative reproducibility score against a fixed threshold — the article accepts reproduced metrics by qualitative comparison with Sections 2.4 and 3.6 (expert review), with no numeric cutoff

## Inputs

- sk (gene-weight) vectors of metagenes from ICA decomposition of dataset A, as a pandas DataFrame
- sk (gene-weight) vectors of metagenes from ICA decomposition of dataset B, as a pandas DataFrame

## Outputs

- rbh_edge_list.csv — edge list of reciprocal best hit ('orthologous') metagene pairs
- rbh_network_metrics.csv — table with modularity, average clustering coefficient, and community membership of each metagene

## How to apply

Load the sk (gene-weight) vectors of each ICA decomposition into pandas DataFrames and compute pairwise correlations between the sk vectors of dataset A and dataset B to obtain a metagene-by-metagene similarity matrix. For each metagene in each dataset, find its best-correlated counterpart in the other dataset, and retain only reciprocal best hits (RBH) as edges of 'orthologous' metagenes — this symmetry requirement filters unstable, one-way matches. Build an undirected graph from the RBH edge list with networkx and compute the modularity from community detection, the average clustering coefficient, and each metagene's community membership. Write the RBH edge list to rbh_edge_list.csv and the summary metrics table to rbh_network_metrics.csv. The RBH graph structure (community count, typical community sizes, clustering) serves as the reproducibility assessment of the components across the two decompositions.

## Related tools

- **Python** (Implementation language for the RBH matching, graph construction, and metric computation (notebook runs on Python 3.7.3)) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Jupyter notebook** (Interactive notebook accompanying the review that illustrates the RBH metagene-matching workflow) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **pandas** (Loads the sk gene-weight vectors of both ICA decompositions into DataFrames for pairwise correlation)
- **networkx** (Builds the undirected RBH graph from the edge list and computes modularity, average clustering coefficient, and community membership)

## Examples

```
import networkx as nx; G = nx.from_pandas_edgelist(rbh_edges, 'metagene_A', 'metagene_B'); print(nx.modularity(G, nx.community.greedy_modularity_communities(G)), nx.average_clustering(G))
```

## Evaluation signals

- rbh_edge_list.csv exists and every edge is a reciprocal best hit (each pair is the argmax correlation in both directions)
- rbh_network_metrics.csv exists and contains modularity, average clustering coefficient, and community columns
- Each metagene in the RBH graph participates in at most one edge per decomposition (RBH is a matching, not a general similarity graph)
- Reproduced metrics qualitatively match those reported in Sections 2.4 and 3.6 of the article, per expert review
- Community count and typical community sizes are consistent with the number of components retained in each ICA decomposition

## Limitations

- No quantitative threshold for acceptance — reproduced metrics are judged by qualitative comparison with Sections 2.4 and 3.6 since the section text provides no numeric cutoff
- ICA decomposition is stochastic; matched metagenes can shift between runs, so the RBH graph reflects run-specific stability rather than exact component identity

## Evidence

- [methods] card_finding: [evidence span withheld — anchor status: not_found]
- [methods] card_validation: [evidence span withheld — anchor status: not_found]
- [abstract] abstract_scope: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components"
- [readme] readme_notebook: "Jupyter Notebook containing example of Independent Component Analysis (ICA) accompanying the review in Internal Journal of Molecular Sciences (IJMS)"
