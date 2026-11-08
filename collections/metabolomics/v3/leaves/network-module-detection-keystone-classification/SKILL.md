---
name: network-module-detection-keystone-classification
description: Use when a co-occurrence network (e.g. MetaNet c_net_build output at
  r_threshold=0.6, p=0.05) needs community decomposition and hub/connector classification,
  such as microbial species networks from abundance tables.
when_to_use_negative:
- The network was built with a different similarity metric or threshold, so Zi/P keystone
  cutoffs and module stability are not comparable to the article's Spearman |ρ|>0.6
  standard
- The graph is too small or too sparse (e.g. the singletons already pruned by delete_single=TRUE)
  to support meaningful community detection or keystone-role statistics
- The goal is differential or cross-condition network comparison (c_net_union/intersect/difference)
  rather than within-network module and keystone analysis
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_3300
tools:
- name: MetaNet
  role: Performs module detection (c_net_module/get_module), keystone role classification
    (zp_analyse/zp_plot), and annotated module visualization (c_net_plot with plot_module/mark_module)
  repo: https://github.com/Asa12138/MetaNet
- name: igraph
  role: Underlying graph library providing the community-detection algorithms wrapped
    by c_net_module and degree/topology calculations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:8cef66a8a07aed3381f7018922be4c022f5cf1928ab7a5244a1c354ee5db2371
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - R
  - ggraph
  - igraph
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# network-module-detection-keystone-classification

## Summary

Detects densely connected modules in an omics co-occurrence network built with MetaNet (c_net_module/get_module) and classifies keystone taxa by within-module (Zi) and among-module (P) connectivity, using Spearman-|ρ|>0.6 networks as the substrate.

## When to use

Use when a co-occurrence network (e.g. MetaNet c_net_build output at r_threshold=0.6, p=0.05) needs community decomposition and hub/connector classification, such as microbial species networks from abundance tables.

## When NOT to use

- The network was built with a different similarity metric or threshold, so Zi/P keystone cutoffs and module stability are not comparable to the article's Spearman |ρ|>0.6 standard
- The graph is too small or too sparse (e.g. the singletons already pruned by delete_single=TRUE) to support meaningful community detection or keystone-role statistics
- The goal is differential or cross-condition network comparison (c_net_union/intersect/difference) rather than within-network module and keystone analysis

## Inputs

- MetaNet network object (metanet/igraph) built from a correlation matrix (e.g. Spearman with BH-adjusted p-values, |r|>0.6, p<0.05)
- Species/feature abundance table (e.g. Case 1 `micro` table, 914 prevalent species) used to build the network
- Optional vertex annotation tables (taxonomy, abundance) attached via c_net_set for plotting

## Outputs

- Network object annotated with a `module` vertex attribute assigning each node to a community (6 modules in Case 1)
- Zi–P classification per node identifying module hubs and connectors (13 hubs, 19 connectors in Case 1)
- Module composition statistics (links_stat on module counts) and annotated c_net_plot figure with marked modules

## How to apply

Start from the network assembled by c_net_calculate (Spearman correlations, BH-adjusted p-values) and c_net_build (r_threshold=0.6, p_threshold=0.05, use_p_adj=TRUE, delete_single=TRUE) — for the Case 1 underwater exposome data this yields 871 species nodes after filtering to taxa present in >10% of samples. Run module detection with c_net_module/get_module, which wraps multiple community-detection algorithms in igraph, to partition the network (Case 1 resolved six modules). Then apply zp_analyse/zp_plot to compute each node's within-module connectivity Zi and among-module connectivity P, classifying module hubs and connectors from the Zi–P plane (13 module hubs and 19 connectors in Case 1). Visualize with plot_module/mark_module options in c_net_plot or chord/Sankey summaries via links_stat. Verify topology context by fitting a power law to the degree distribution (fit_power) and computing global indices with net_par, since keystone interpretation is meaningful only for scale-free, modular networks.

## Related tools

- **MetaNet** (Performs module detection (c_net_module/get_module), keystone role classification (zp_analyse/zp_plot), and annotated module visualization (c_net_plot with plot_module/mark_module)) — https://github.com/Asa12138/MetaNet
- **igraph** (Underlying graph library providing the community-detection algorithms wrapped by c_net_module and degree/topology calculations)

## Examples

```
net <- c_net_build(c_net_calculate(totu), r_threshold = 0.6, p_threshold = 0.05); net_mod <- c_net_module(net); zp <- zp_analyse(net_mod); c_net_plot(net_mod, plot_module = TRUE, mark_module = TRUE)
```

## Evaluation signals

- Node count and module count match the reproducibility target: 871 species nodes partitioned into 6 modules for the Case 1 network (vs 914 input taxa after >10% prevalence filtering)
- Keystone-role counts match the reported result: 13 module hubs and 19 connectors identified from Zi–P connectivity
- Degree distribution fits a power law (fit_power), confirming the scale-free topology that the article reports for the Case 1 network
- Every node carries a non-empty `module` attribute and each Zi/P value is finite, so no node is unassigned or dropped by the module step
- The annotated plot renders with marked modules and no orphan vertices, and links_stat sums over modules equal the total edge count of the built network

## Limitations

- Correlation-based network construction may overlook complex, non-linear, or indirect associations, so keystone roles reflect linear co-occurrence only
- Lack of standardized protocols for similarity metrics, thresholding, and null model selection means module and keystone results are sensitive to the chosen r_threshold/p_threshold and community algorithm
- MetaNet README notes the network must be built with p-value adjustment awareness ('Have not do p-value adjustment! use the p.value to build network.'), so skipping BH adjustment can inflate edges and distort module structure

## Evidence

- [readme] c_net_module includes multiple community detection algorithms: [evidence span withheld — anchor status: external_doc]
- [other] network built at r_threshold 0.6 with BH-adjusted p-values and 871 nodes, 6 modules, 13 hubs, 19 connectors: [evidence span withheld — anchor status: not_found]
- [other] keystone classification via Zi and P connectivity: [evidence span withheld — anchor status: not_found]
- [readme] modules visualized with marked modules in c_net_plot: [evidence span withheld — anchor status: not_found]
