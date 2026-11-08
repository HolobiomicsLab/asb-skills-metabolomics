---
name: network-modularity-and-clustering-analysis
description: Use when two ICA decompositions (metagene/sk weight vectors) from different
  transcriptomic datasets must be matched to assess reproducibility of independent
  components via a graph of 'orthologous' metagenes.
when_to_use_negative:
- You need quantitative agreement thresholds between components — the section provides
  no numeric cutoff, so reproducibility is judged only by qualitative/expert comparison
  with Sections 2.4 and 3.6.
- The decomposition outputs are sample-weight (dr) vectors or single-dataset components
  with no second decomposition to match against.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implementation language for correlation computation, RBH extraction, and graph
    metrics
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Jupyter notebook
  role: Interactive notebook environment accompanying the review that illustrates
    the ICA analysis concepts and workflow
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:e58947733db50834498be147db37d2567a8ce6af099e3938fcb5d9f92ffc4566
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# network-modularity-and-clustering-analysis

## Summary

Reconstruct a reciprocal best hit (RBH) graph linking metagenes (sk vectors) from two ICA decompositions of cancer transcriptomic datasets, then quantify component reproducibility via modularity, average clustering coefficient, and community structure computed with networkx.

## When to use

Use when two ICA decompositions (metagene/sk weight vectors) from different transcriptomic datasets must be matched to assess reproducibility of independent components via a graph of 'orthologous' metagenes.

## When NOT to use

- You need quantitative agreement thresholds between components — the section provides no numeric cutoff, so reproducibility is judged only by qualitative/expert comparison with Sections 2.4 and 3.6.
- The decomposition outputs are sample-weight (dr) vectors or single-dataset components with no second decomposition to match against.

## Inputs

- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset A (pandas DataFrame)
- sk (gene-weight) vectors of ICA decomposition for cancer omics dataset B (pandas DataFrame)

## Outputs

- rbh_edge_list.csv (RBH edge list of matched metagenes)
- rbh_network_metrics.csv (modularity, average clustering coefficient, community membership per metagene)
- networkx undirected RBH graph of 'orthologous' metagenes

## How to apply

Load the sk (gene-weight) vectors of the two ICA decompositions into pandas DataFrames and compute pairwise correlations between sk vectors of dataset A and dataset B to obtain a metagene-by-metagene similarity matrix. For each metagene, keep only its best-correlated counterpart in the other dataset, retaining reciprocal best hits (RBH) as edges of 'orthologous' metagenes. Build an undirected graph from the RBH edge list with networkx, then run community detection to derive modularity and community membership, and compute the average clustering coefficient. Write the edge list to rbh_edge_list.csv and the summary metrics table to rbh_network_metrics.csv, and validate by qualitative comparison with the metrics reported in Sections 2.4 and 3.6 of the review (expert review), since no quantitative threshold is given in the section text.

## Related tools

- **Python** (Implementation language for correlation computation, RBH extraction, and graph metrics) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Jupyter notebook** (Interactive notebook environment accompanying the review that illustrates the ICA analysis concepts and workflow) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
import pandas as pd, networkx as nx; S = A_sk.T.dot(B_sk); rbh = [(a,b) for a in A_sk for b in [A_sk[a].idxmax()] if B_sk[b].idxmax()==a]; G = nx.Graph(rbh)
```

## Evaluation signals

- rbh_edge_list.csv and rbh_network_metrics.csv both exist after the run.
- rbh_network_metrics.csv contains modularity, average clustering coefficient, and community columns.
- Every edge in rbh_edge_list.csv is a reciprocal best hit: each metagene's counterpart is also its counterpart's best hit.
- Community assignment covers each metagene in the RBH graph (community membership is defined for every node).
- Reproduced modularity and average clustering coefficient qualitatively match the values reported in Sections 2.4 and 3.6 (expert review, no numeric threshold provided).

## Limitations

- The reproduced metrics are accepted by qualitative comparison with Sections 2.4 and 3.6; the section text provides no quantitative threshold.
- Matching depends on the ICA decomposition being stable; reproducibility assessment assumes the sk vectors from two independent decompositions of different transcriptomic datasets are comparable.
- The RBH criterion keeps only the single best-correlated counterpart per metagene, so secondary correlations are discarded.

## Evidence

- [methods] Metagenes from two ICA decompositions are matched as reciprocal best hits to build an RBH graph: [evidence span withheld — anchor status: not_found]
- [methods] Build graph with networkx and compute metrics: [evidence span withheld — anchor status: not_found]
- [methods] Output files and validation: [evidence span withheld — anchor status: not_found]
- [readme] Notebook purpose and repo: "Jupyter Notebook containing example of Independent Component Analysis (ICA) accompanying the review in Internal Journal of Molecular Sciences (IJMS)"
- [intro] Tooling context from article: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
