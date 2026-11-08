---
name: fdr-multiple-testing-correction
description: Use when a metabolomics or multi-omics analysis yields thousands of pairwise
  correlation P-values (e.g., metabolite-protein Spearman correlations across ~202
  shared samples) and edges must be called at FDR < 0.05 with an effect-size cutoff
  such as |r| > 0.3.
when_to_use_negative:
- The analysis performs a single hypothesis test or a small pre-specified comparison
  set, where per-test P-value thresholds suffice
- Raw data have not yet been filtered (samples with >90% zeros, variables with >40%
  zeros) or normalized to counts per million, so correlation P-values are computed
  on unreliable zero-inflated variables
- The goal is building weighted co-occurrence networks without an explicit edge-significance
  criterion rather than a thresholded correlation network
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server whose network module performs correlation-based network construction
    with editable parameters, including FDR-thresholded edge selection and downstream
    centrality analysis
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Predecessor desktop platform whose R backend provides the correlation-network
    scripts for iMAP workflows
  repo: https://github.com/IP4M
- name: Cytoscape.js
  role: Visualization of the FDR-filtered edge table and computed node centralities
    as an interactive network
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:98de4231850c8c737ee47457ca6f231518a4c6909de57164284250d8827a2900
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape.js
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fdr-multiple-testing-correction

## Summary

Apply false discovery rate (FDR) correction to the large family of pairwise P-values generated in correlation-based network analysis, so that qualified edges (FDR < 0.05, |r| > 0.3) can be selected at a controlled error rate. In iMAP this step underpins the network module built on the 308-sample IBD stool proteomics/metabolomics dataset.

## When to use

Use when a metabolomics or multi-omics analysis yields thousands of pairwise correlation P-values (e.g., metabolite-protein Spearman correlations across ~202 shared samples) and edges must be called at FDR < 0.05 with an effect-size cutoff such as |r| > 0.3.

## When NOT to use

- The analysis performs a single hypothesis test or a small pre-specified comparison set, where per-test P-value thresholds suffice
- Raw data have not yet been filtered (samples with >90% zeros, variables with >40% zeros) or normalized to counts per million, so correlation P-values are computed on unreliable zero-inflated variables
- The goal is building weighted co-occurrence networks without an explicit edge-significance criterion rather than a thresholded correlation network

## Inputs

- Processed metabolomics abundance matrix (234 metabolites x 202 shared IBD stool samples, counts-per-million normalized)
- Processed proteomics abundance matrix (112 proteins x 202 shared samples, counts-per-million normalized)
- Matrix of raw pairwise Spearman correlation P-values for all metabolite-protein pairs

## Outputs

- FDR-adjusted P-value (q-value) vector/matrix aligned to the raw correlation P-values
- Filtered edge table (correlation_network_edge_table.csv) with columns r, P, FDR, -ln(P), -ln(FDR), |r| containing the 95 qualified edges
- Node table derived from the edge table with graph-theory centralities (degree, betweenness, closeness, eigenvector)

## How to apply

After separately filtering each omics matrix (remove samples with >90% zeros and variables with >40% zeros, retain the 202 samples shared across datasets, normalize each sample to counts per million, yielding 234 metabolites and 112 proteins), compute pairwise Spearman correlation coefficients and raw P-values for all metabolite-protein variable pairs. Convert the P-value matrix to FDR-adjusted values (e.g., Benjamini-Hochberg via p.adjust) before any edge calling, because the 234x112 pair space inflates the raw significance count. Retain only pairs passing FDR < 0.05 and |r| > 0.3 as network edges; in the iMAP IBD application this produces exactly 95 qualified correlations. Write the edge table with columns r, P, FDR, -ln(P), -ln(FDR), and |r| so downstream node centralities (degree, betweenness, closeness, eigenvector) can be computed to identify hub proteins such as glutamate dehydrogenase [NAD(P)(+)] and hub metabolites such as adipate and 3-methyladipate/pimelate.

## Related tools

- **iMAP** (Web server whose network module performs correlation-based network construction with editable parameters, including FDR-thresholded edge selection and downstream centrality analysis) — https://imap.metaboprofile.cloud/
- **IP4M** (Predecessor desktop platform whose R backend provides the correlation-network scripts for iMAP workflows) — https://github.com/IP4M
- **Cytoscape.js** (Visualization of the FDR-filtered edge table and computed node centralities as an interactive network)

## Examples

```
Rscript -e 'p <- cor.test(met[i,], prot[j,], method="spearman")$p.value; q <- p.adjust(pvals, method="BH")'
```

## Evaluation signals

- The filtered edge table contains exactly 95 qualified edges at the FDR < 0.05 and |r| > 0.3 thresholds on the processed IBD data
- Edge table schema includes all required columns: r, P, FDR, -ln(P), -ln(FDR), |r|, with every FDR value <= 0.05 and every |r| > 0.3
- Retained variable counts before testing match the report: 234 metabolites and 112 proteins over 202 shared samples
- Computed centralities identify glutamate dehydrogenase [NAD(P)(+)] and 6-phosphofructokinase as the largest protein nodes, and adipate and 3-methyladipate/pimelate as the largest metabolite nodes, consistent with Figure 5
- Monotonicity check: every raw P retained in the edge table has FDR(P) >= P, and -ln(P)/-ln(FDR) columns are non-negative

## Limitations

- Threshold sensitivity: the 95-edge result is specific to FDR < 0.05 and |r| > 0.3; other cutoffs change the network and the hub identification
- iMAP does not implement raw data preprocessing or peak picking/annotation, so the input matrices must already be quality-controlled before correlation testing
- Spearman correlations on zero-inflated, sparsely-detected clinical stool data can be unstable if the >90%/>40% zero filters and counts-per-million normalization are skipped

## Evidence

- [abstract] A new module was incorporated with sufficient editable parameters for network construction, visualization, and analysis.: "A new module was incorporated with sufficient editable parameters for network construction, visualization, and analysis."
- [methods] Samples with >90% zeros and variables with >40% zeros were removed. Moreover, the data in each sample were normalized to counts per million: [evidence span withheld — anchor status: not_found]
- [intro] New modules for predictive model building and validation and correlation-based network construction and analysis have been added: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [other] Filter correlation pairs at FDR < 0.05 and |r| > 0.3, keeping the 95 qualified correlations as network edges: [evidence span withheld — anchor status: not_found]
