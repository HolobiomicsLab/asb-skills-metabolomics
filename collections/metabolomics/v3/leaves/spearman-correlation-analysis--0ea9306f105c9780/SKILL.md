---
name: spearman-correlation-analysis
description: Use when two processed abundance matrices (e.g., metabolomics and proteomics
  from the same samples) must be integrated via pairwise Spearman correlations to
  construct a metabolite-protein network and identify hub nodes.
when_to_use_negative:
- Input is raw spectral data (mzML etc.) — iMAP does not implement raw data preprocessing
  or peak picking and annotation modules
- The goal is differential-abundance testing between phenotypes rather than pairwise
  variable-variable association network construction
- The two omics datasets lack shared samples, so paired metabolite-protein correlation
  cannot be computed
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: iMAP
  role: Web server whose network module performs Spearman correlation-based network
    construction, visualization, and analysis with editable parameters
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Desktop precursor platform covering the metabolomics data-analysis steps;
    iMAP's backend R scripts extend its workflows
  repo: https://github.com/IP4M
- name: Cytoscape.js
  role: Used to visualize the qualified correlation network built from the Spearman
    edge table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:cf214b05e1648fa1b3ccf68d805d6d4db25c6ec68d1d39d11e51906eb5c91fdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape.js
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spearman-correlation-analysis

## Summary

Computes pairwise Spearman rank correlations between metabolite and protein abundance variables to build a correlation-based network, using iMAP's network module with editable thresholds to qualify edges (FDR < 0.05, |r| > 0.3) and derive graph-theory centralities.

## When to use

Use when two processed abundance matrices (e.g., metabolomics and proteomics from the same samples) must be integrated via pairwise Spearman correlations to construct a metabolite-protein network and identify hub nodes.

## When NOT to use

- Input is raw spectral data (mzML etc.) — iMAP does not implement raw data preprocessing or peak picking and annotation modules
- The goal is differential-abundance testing between phenotypes rather than pairwise variable-variable association network construction
- The two omics datasets lack shared samples, so paired metabolite-protein correlation cannot be computed

## Inputs

- Processed metabolomics abundance matrix (zeros-filtered, counts-per-million normalized), e.g., 234 metabolites x 202 samples
- Processed proteomics abundance matrix (zeros-filtered, counts-per-million normalized), e.g., 112 proteins x 202 samples
- FDR and |r| thresholds (e.g., FDR < 0.05, |r| > 0.3)

## Outputs

- correlation_network_edge_table.csv with columns r, P, FDR, -ln(P), -ln(FDR), |r| (e.g., 95 qualified edges)
- Network node table with graph-theory centralities (degree, betweenness, closeness, eigenvector) per node
- Correlation-based metabolite-protein network for visualization (iMAP / Cytoscape.js)

## How to apply

First preprocess each omics matrix in iMAP's data-processing module: remove samples with >90% zeros and variables with >40% zeros, retain only shared samples between datasets, and normalize each sample to counts per million. Compute pairwise Spearman correlation coefficients between all metabolite-protein variable pairs on the processed data (e.g., 234 metabolites vs 112 proteins across 202 shared IBD stool samples). Filter correlation pairs at FDR < 0.05 and |r| > 0.3 to qualify network edges, and emit an edge table with r, P, FDR, -ln(P), -ln(FDR), and |r| columns. Generate the node table from the edge table and compute degree, betweenness, closeness, and eigenvector centralities to rank hub proteins and hub metabolites; the network can be exported for visualization in iMAP or Cytoscape.js.

## Related tools

- **iMAP** (Web server whose network module performs Spearman correlation-based network construction, visualization, and analysis with editable parameters) — https://imap.metaboprofile.cloud/
- **IP4M** (Desktop precursor platform covering the metabolomics data-analysis steps; iMAP's backend R scripts extend its workflows) — https://github.com/IP4M
- **Cytoscape.js** (Used to visualize the qualified correlation network built from the Spearman edge table)

## Evaluation signals

- Edge table contains exactly the expected number of qualified edges at the chosen thresholds (e.g., 95 edges at FDR < 0.05 and |r| > 0.3)
- All edge rows carry complete r, P, FDR, -ln(P), -ln(FDR), and |r| values
- Preprocessing retains the expected variable counts (e.g., 234 metabolites and 112 proteins) and the shared-sample count (e.g., 202 samples)
- Centrality ranking identifies the known hub nodes — glutamate dehydrogenase [NAD(P)(+)] and 6-phosphofructokinase as largest protein nodes, adipate and 3-methyladipate/pimelate as largest metabolite nodes — consistent with Figure 5
- Every edge in the network satisfies both the FDR < 0.05 and |r| > 0.3 cutoffs (no unqualified edges leak through)

## Limitations

- iMAP does not implement raw data preprocessing or peak picking and annotation; inputs must already be quantified feature tables
- Network construction depends on user-editable thresholds, so results are reproducible only if the FDR and |r| cutoffs and the zero-removal filters are fixed and reported
- Only samples shared across omics datasets can be used, so filtering one dataset changes the paired sample set available for correlation

## Evidence

- [abstract] A new network module was incorporated with sufficient editable parameters for network construction, visualization, and analysis.: "A new module was incorporated with sufficient editable parameters for network construction, visualization, and analysis."
- [methods] Samples with >90% zeros and variables with >40% zeros were removed. Moreover, the data in each sample were normalized to counts per million: [evidence span withheld — anchor status: not_found]
- [intro] New modules for predictive model building and validation and correlation-based network construction and analysis have been added: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [methods] The backend R scripts are available at the iMAP GitHub repository: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] iMAP is freely available for academic usage (License MPL 2.0).: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
