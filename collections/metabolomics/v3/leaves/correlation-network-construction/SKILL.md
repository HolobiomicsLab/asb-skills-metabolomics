---
name: correlation-network-construction
description: Use when raw feature abundance or count matrices (OTU/species/metabolite/gene
  tables, feature names as columns) need to be converted into an edge-weighted network
  via correlation thresholds, especially for multi-omics integration.
when_to_use_negative:
- Input is already an edge list or adjacency matrix — skip construction and go straight
  to c_net_plot/topology analysis.
- Data are similarity scores from curated databases (e.g. STRING/KEGG hybrid networks)
  rather than pairwise correlation of a feature table.
- The goal is clustering of samples or ordination, not feature-to-feature network
  inference.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: MetaNet
  role: computes pairwise correlations (c_net_calculate) and builds the filtered igraph
    network (c_net_build / multi_net_build)
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: runtime environment for MetaNet; requires R >= 4.0
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: core data structure underlying MetaNet network objects; used for degree, neighbors,
    and layout
- name: pcutils
  role: dependency providing example data (otutab, multi_test) and utilities like
    t2 used before c_net_calculate
  repo: https://github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:bd33314d08e7ee8a6bfbccfd374974b101491459d05bf6b646c3fe2fba1998f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - NetCoMi
  - R
  - WGCNA
  - ggClusterNet
  - igraph
  - microeco
  - pcutils
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# correlation-network-construction

## Summary

Builds co-occurrence or cross-layer omics networks by computing pairwise correlation coefficients and p-values (e.g. with MetaNet's c_net_calculate) and then filtering edges by significance and correlation thresholds with c_net_build before visualization or topology analysis.

## When to use

Use when raw feature abundance or count matrices (OTU/species/metabolite/gene tables, feature names as columns) need to be converted into an edge-weighted network via correlation thresholds, especially for multi-omics integration.

## When NOT to use

- Input is already an edge list or adjacency matrix — skip construction and go straight to c_net_plot/topology analysis.
- Data are similarity scores from curated databases (e.g. STRING/KEGG hybrid networks) rather than pairwise correlation of a feature table.
- The goal is clustering of samples or ordination, not feature-to-feature network inference.

## Inputs

- feature abundance/count matrix (features x samples; e.g. otutab or totu transposed table)
- list of omics-layer matrices for multi-omics construction (e.g. micro, metab, transc)
- correlation method name and p-value adjustment method
- edge filtering thresholds (r_threshold, p_threshold)

## Outputs

- correlation matrix (pairwise coefficients and p-values)
- igraph/metanet network object with vertex and edge attributes (e.g. v_group, e_class)
- edge table with correlation sign, r and p values per edge

## How to apply

Transpose the feature table so that features are columns, then call c_net_calculate(totu) (optionally with method, p.adjust.method, and threads arguments) to obtain a correlation matrix whose vectorized computation scales to >10,000 features. Build the network with c_net_build(cor, r_threshold = ..., p_threshold = ..., use_p_adj = ...), removing singletons via delete_single; note MetaNet warns 'Have not do p-value adjustment! use the p.value to build network' unless adjustment is requested, so set use_p_adj explicitly when multiple-testing control matters. For multi-omics integration use multi_net_build(list(Microbiome=..., Metabolome=..., Transcriptome=...), r_threshold = 0.6), which automatically matches samples across layers and computes cross-layer correlations in one call. Annotate vertices with c_net_set (e.g. vertex_class = c("Phylum","kingdom","type"), vertex_size from colSums abundance) and optionally restrict the subnetwork with c_net_filter. Correctness is judged by whether edges retained meet the stated r_threshold and significance filter, whether all samples matched across omics layers, and whether the resulting igraph object carries vertex_group and e_class annotations.

## Related tools

- **MetaNet** (computes pairwise correlations (c_net_calculate) and builds the filtered igraph network (c_net_build / multi_net_build)) — https://github.com/Asa12138/MetaNet
- **R** (runtime environment for MetaNet; requires R >= 4.0) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (core data structure underlying MetaNet network objects; used for degree, neighbors, and layout)
- **pcutils** (dependency providing example data (otutab, multi_test) and utilities like t2 used before c_net_calculate) — https://github.com/Asa12138/pcutils

## Examples

```
library(pcutils); library(MetaNet); totu <- t2(otutab[1:70, ]); cor <- c_net_calculate(totu); net <- c_net_build(cor, r_threshold = 0.65)
```

## Evaluation signals

- Every retained edge satisfies the stated r_threshold (e.g. r >= 0.65 in the quick start) and the applied p-value filter; singletons removed when delete_single is set.
- No 'Have not do p-value adjustment!' warning if use_p_adj/adjustment was intended; the built network reflects the chosen multiple-testing correction.
- For multi-omics builds, the log reports 'All samples matched.' and 'All features are OK.' across layers before correlation computation.
- Edge sign distribution matches expectation (e.g. microbial edges predominantly positive, chemical edges predominantly negative in the exposome-transcriptome case study).
- Network object is a valid igraph/metanet graph with populated v_group, e_class, and abundance-based vertex_size attributes.

## Limitations

- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation.
- Correlation-based networks may overlook complex, non-linear, or indirect associations.
- MetaNet warns when p-value adjustment has not been applied — using raw p-values risks spurious edges in large feature sets.

## Evidence

- [readme] cor-based network construction is central: "MetaNet enables fast and scalable correlation-based network construction for datasets with more than 10,000 features"
- [readme] quick start pipeline: "cor <- c_net_calculate(totu)
net <- c_net_build(cor, r_threshold = 0.65)"
- [readme] multi-omics construction: [evidence span withheld — anchor status: not_found]
- [intro] threshold bias caveat: [evidence span withheld — anchor status: not_found]
- [intro] computational demand: "computational performance is often a bottleneck—especially for correlation-based network construction on high-dimensional datasets, which may require hours or even days25."
- [results] task_001 benchmark workflow: [evidence span withheld — anchor status: not_found]
