---
name: correlation-network-edge-construction
description: Use when you have two processed abundance matrices (e.g., metabolomics
  and proteomics) from the same samples and need to build a correlation-based network
  identifying hub metabolites and hub proteins.
when_to_use_negative:
- Input data still contain missing values or unnormalized counts — the module expects
  data already filtered (samples with >90% zeros and variables with >40% zeros removed)
  and normalized to counts per million.
- You need raw data preprocessing, peak picking, or annotation — iMAP did not implement
  these modules and they must be done upstream.
- The two omics datasets do not share samples; the workflow requires retaining only
  the samples common to both datasets before correlation computation.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: iMAP
  role: Web server whose network module performs correlation-based network construction,
    visualization, and analysis with editable parameters
  repo: https://github.com/IP4M
- name: IP4M
  role: Backend R scripts implementing iMAP's analysis workflows including network
    construction
  repo: github.com/IP4M/R-scripts-for-iMAP
- name: Cytoscape.js
  role: Used for network visualization of the constructed metabolite-protein correlation
    network
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:de7d794d778fc5b59769ab53e5a8beb256c2256370fd0de53d9ff5ad3afde819
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape.js
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# correlation-network-edge-construction

## Summary

Constructs a metabolite-protein correlation network by computing pairwise Spearman correlations between processed multi-omics variables and filtering them at FDR < 0.05 and |r| > 0.3, as demonstrated with iMAP's network module on IBD stool sample data.

## When to use

Use when you have two processed abundance matrices (e.g., metabolomics and proteomics) from the same samples and need to build a correlation-based network identifying hub metabolites and hub proteins.

## When NOT to use

- Input data still contain missing values or unnormalized counts — the module expects data already filtered (samples with >90% zeros and variables with >40% zeros removed) and normalized to counts per million.
- You need raw data preprocessing, peak picking, or annotation — iMAP did not implement these modules and they must be done upstream.
- The two omics datasets do not share samples; the workflow requires retaining only the samples common to both datasets before correlation computation.

## Inputs

- Processed metabolomics abundance matrix (e.g., 234 metabolites x 202 shared IBD stool samples, counts-per-million normalized)
- Processed proteomics abundance matrix (e.g., 112 proteins x 202 shared samples)
- Threshold parameters: FDR cutoff (0.05) and absolute correlation cutoff (0.3)

## Outputs

- correlation_network_edge_table.csv with columns r, P, FDR, -ln(P), -ln(FDR), |r| (95 qualified edges)
- Node table derived from the edge table with graph-theory centralities (degree, betweenness, closeness, eigenvector) per node
- Correlation network visualization (e.g., in Cytoscape.js) highlighting hub proteins and hub metabolites

## How to apply

Process each omics dataset separately: remove samples with >90% zeros and variables with >40% zeros, retain only samples shared between the datasets (202 of 308 IBD stool samples), and normalize each sample to counts per million, verifying 234 metabolites and 112 proteins are retained. Compute pairwise Spearman correlation coefficients between all metabolite-protein variable pairs, then filter pairs at FDR < 0.05 and |r| > 0.3 to keep 95 qualified correlations as network edges. Write the edge table (with r, P, FDR, -ln(P), -ln(FDR), |r|) to correlation_network_edge_table.csv, generate the node table from the edges, and compute graph-theory centralities (degree, betweenness, closeness, eigenvector) to identify hubs such as glutamate dehydrogenase [NAD(P)(+)] and adipate. iMAP's network module provides sufficient editable parameters for this network construction, visualization, and analysis, with results viewable in Cytoscape.js.

## Related tools

- **iMAP** (Web server whose network module performs correlation-based network construction, visualization, and analysis with editable parameters) — https://github.com/IP4M
- **IP4M** (Backend R scripts implementing iMAP's analysis workflows including network construction) — github.com/IP4M/R-scripts-for-iMAP
- **Cytoscape.js** (Used for network visualization of the constructed metabolite-protein correlation network)

## Evaluation signals

- The edge table contains exactly 95 qualified edges at the FDR<0.05 and |r|>0.3 threshold.
- Retained variable counts match the reported 234 metabolites and 112 proteins after filtering.
- The 202 samples shared between metabolomics and proteomics datasets are retained after filtering.
- Centralities identify glutamate dehydrogenase [NAD(P)(+)] and 6-phosphofructokinase as the largest protein nodes and adipate and 3-methyladipate/pimelate as the largest metabolite nodes, consistent with Figure 5 of the paper.
- Edge table columns include r, P, FDR, -ln(P), -ln(FDR), and |r|.

## Limitations

- iMAP did not implement raw data preprocessing or peak picking and annotation modules, so input matrices must be prepared upstream.
- As a web-based tool, data security considerations limited which modules could be implemented server-side.
- Although there are still many limitations, iMAP is an alternative tool — limitations are acknowledged but not fully specified in the article.

## Evidence

- [readme] A new network module was incorporated with sufficient editable parameters for network construction, visualization, and analysis.: "A new network module was incorporated with sufficient editable parameters for network construction, visualization, and analysis."
- [methods] Samples with >90% zeros and variables with >40% zeros were removed.: [evidence span withheld — anchor status: not_found]
- [methods] proteomics and metabolomics data of 308 clinical stool samples (Lloyd-Price et al., 2019) from patients with inflammatory bowel disease: [evidence span withheld — anchor status: not_found]
- [intro] New modules for predictive model building and validation and correlation-based network construction and analysis have been added: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [methods] The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
