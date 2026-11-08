---
name: node-centrality-network-analysis
description: Use when you have two processed abundance matrices (e.g., metabolite
  and protein feature tables across the same samples) and need to build a correlation-based
  network and identify hub nodes via centrality metrics.
when_to_use_negative:
- The two omics datasets do not share samples — the correlation requires the retained
  intersection of samples (202 shared of 308 in the reference), and unmatched samples
  must be dropped first
- Input matrices are raw, unfiltered data — samples with >90% zeros and variables
  with >40% zeros must be removed and data CPM-normalized before correlation, or spurious
  edges will result
- The goal is raw-data preprocessing, peak picking, or metabolite annotation — iMAP
  explicitly did not implement these modules
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server providing the network module with editable parameters for correlation-network
    construction, visualization, and analysis
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: IP4M
  role: Predecessor desktop platform whose R backend scripts implement the network
    and data-processing modules
  repo: https://github.com/IP4M
- name: Cytoscape.js
  role: Visualization of the qualified correlation network edges and node centralities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:055200fec3983be483b825557c2010e402d1a3ac451aa594f7e622e25ab1a781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IP4M/R-scripts-for-iMAP
  tools:
  - Cytoscape.js
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Node-centrality correlation network analysis

## Summary

Constructs a metabolite-protein correlation network from processed multi-omics abundance matrices (Spearman correlation, FDR < 0.05, |r| > 0.3) and ranks nodes by graph-theory centralities (degree, betweenness, closeness, eigenvector) to identify hub metabolites and hub proteins. This reproduces the iMAP network module workflow demonstrated on IBD stool-sample metabolomics and proteomics data.

## When to use

Use when you have two processed abundance matrices (e.g., metabolite and protein feature tables across the same samples) and need to build a correlation-based network and identify hub nodes via centrality metrics.

## When NOT to use

- The two omics datasets do not share samples — the correlation requires the retained intersection of samples (202 shared of 308 in the reference), and unmatched samples must be dropped first
- Input matrices are raw, unfiltered data — samples with >90% zeros and variables with >40% zeros must be removed and data CPM-normalized before correlation, or spurious edges will result
- The goal is raw-data preprocessing, peak picking, or metabolite annotation — iMAP explicitly did not implement these modules

## Inputs

- Processed metabolomics abundance matrix (samples × metabolites; e.g., 202 samples × 234 metabolites, CPM-normalized)
- Processed proteomics abundance matrix (samples × proteins; e.g., 202 samples × 112 proteins, CPM-normalized)
- Correlation thresholds: FDR < 0.05 and |r| > 0.3

## Outputs

- correlation_network_edge_table.csv with columns r, P, FDR, -ln(P), -ln(FDR), |r| (exactly 95 qualified edges in the reference analysis)
- Node table derived from the edge table with degree, betweenness, closeness, and eigenvector centralities per node
- Identified hub proteins and hub metabolites (largest-centrality nodes)

## How to apply

First process each omics dataset separately: remove samples with >90% zeros and variables with >40% zeros, retain only samples shared between datasets (202 of 308 IBD stool samples), and normalize each sample to counts per million, verifying that 234 metabolites and 112 proteins are retained. Compute pairwise Spearman correlation coefficients between all metabolite-protein variable pairs on the processed matrices. Filter correlation pairs at FDR < 0.05 and |r| > 0.3 to keep the qualifying edges (95 in the reference analysis), and write an edge table including r, P, FDR, -ln(P), -ln(FDR), and |r|. Generate the node table from the edge table and compute degree, betweenness, closeness, and eigenvector centralities per node; the largest-centrality nodes are the hubs (e.g., glutamate dehydrogenase [NAD(P)(+)] and 6-phosphofructokinase among proteins, adipate and 3-methyladipate/pimelate among metabolites). iMAP's network module supplies editable parameters for this network construction, visualization, and analysis, and results can be exported for visualization in Cytoscape.js.

## Related tools

- **iMAP** (Web server providing the network module with editable parameters for correlation-network construction, visualization, and analysis) — https://github.com/IP4M/R-scripts-for-iMAP
- **IP4M** (Predecessor desktop platform whose R backend scripts implement the network and data-processing modules) — https://github.com/IP4M
- **Cytoscape.js** (Visualization of the qualified correlation network edges and node centralities)

## Examples

```
m <- cor(t(metab_cpm), t(prot_cpm), method="spearman"); edges <- which(p.adjust(m, "BH") < 0.05 & abs(r) > 0.3, arr.ind=TRUE)
```

## Evaluation signals

- The edge table contains exactly 95 qualified edges at the FDR < 0.05 and |r| > 0.3 threshold
- The processed data retain 234 metabolites and 112 proteins after the >90% zeros (samples) / >40% zeros (variables) filters and restriction to the 202 shared samples
- Centrality computation identifies glutamate dehydrogenase [NAD(P)(+)] and 6-phosphofructokinase as the largest protein nodes
- Centrality computation identifies adipate and 3-methyladipate/pimelate as the largest metabolite nodes, consistent with Figure 5 of the paper
- Every edge row carries complete r, P, FDR, -ln(P), -ln(FDR), and |r| values with no missing entries

## Limitations

- Raw data preprocessing and peak picking and annotation modules were not implemented in iMAP, so inputs must already be feature tables
- Network thresholds are editable, so reproducing the reference 95-edge result requires exactly FDR < 0.05 and |r| > 0.3 on the identically processed data
- Node metrics depend on the correlation-pair filter; changing zero-removal or normalization steps changes the retained edges and hub rankings

## Evidence

- [abstract] Network module with editable parameters: "A new module was incorporated with sufficient editable parameters for network construction, visualization, and analysis."
- [methods] Zero-removal filters and CPM normalization on IBD data: [evidence span withheld — anchor status: not_found]
- [methods] Spearman correlation, edge thresholds, and hub identification on IBD stool data: [evidence span withheld — anchor status: not_found]
- [methods] Centralities identify hub proteins and metabolites: [evidence span withheld — anchor status: not_found]
