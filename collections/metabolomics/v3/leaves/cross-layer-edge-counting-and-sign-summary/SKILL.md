---
name: cross-layer-edge-counting-and-sign-summary
description: Use when a built multi-omics network (e.g. microbiome x transcriptome,
  chemicals x genes) must be summarized per layer pair, to test whether exposure classes
  show distinct correlation-sign patterns and to report node/edge counts per layer
  pair.
when_to_use_negative:
- The network has not yet been built or filtered (no correlation threshold/p-value
  adjustment applied) — run c_net_calculate and c_net_build first
- The question concerns intra-layer topology (module detection, degree centrality)
  rather than cross-layer association counts and sign patterns
- Associations are non-correlation-based (e.g. non-linear or directed causal edges),
  which the article notes correlation-based network optimization may overlook
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: MetaNet
  role: Provides c_net_calculate, c_net_build, get_e/get_v, links_stat, plot_e_type_bar,
    and net_par for building the network and summarizing cross-layer edge counts and
    sign distributions
  repo: github.com/Asa12138/MetaNet
- name: pcutils
  role: Companion package loaded alongside MetaNet; supplies test data (otutab) and
    utilities used in the workflow environment
  repo: github.com/Asa12138/pcutils
- name: igraph
  role: Core dependency underlying MetaNet's network objects, edge extraction, and
    topological operations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:69134e4e39b43961c7aa04209680710d3e0cd658cd48700db6e887a41021b40e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-layer-edge-counting-and-sign-summary

## Summary

Quantify how many nodes and edges link each pair of omics layers in a MetaNet multi-omics network and summarize the sign (positive/negative) distribution of cross-layer correlations, as demonstrated in the Case 2 exposome-transcriptome analysis.

## When to use

Use when a built multi-omics network (e.g. microbiome x transcriptome, chemicals x genes) must be summarized per layer pair, to test whether exposure classes show distinct correlation-sign patterns and to report node/edge counts per layer pair.

## When NOT to use

- The network has not yet been built or filtered (no correlation threshold/p-value adjustment applied) — run c_net_calculate and c_net_build first
- The question concerns intra-layer topology (module detection, degree centrality) rather than cross-layer association counts and sign patterns
- Associations are non-correlation-based (e.g. non-linear or directed causal edges), which the article notes correlation-based network optimization may overlook

## Inputs

- Built MetaNet multi-omics network object (igraph-based) with cross-layer edges and vertex_group attributes
- Correlation matrix from c_net_calculate across omics layers (e.g. 35,587 genes, 2,955 microbial species, 3,729 chemical features)
- Edge sign/metadata attributes (positive vs negative correlation)

## Outputs

- Per layer-pair edge counts (cross-layer association summary)
- Node counts per omics layer that participate in cross-layer edges (e.g. 590 taxa->1,983 genes; 245 chemicals->1,026 genes)
- Edge sign distribution summary/plot (e.g. plot_e_type_bar output)
- CSV of network statistics (e.g. case2_network_stats.csv)

## How to apply

After building the integrated network with c_net_calculate and c_net_build (with r_threshold, p_threshold, use_p_adj, and delete_single filtering), extract cross-layer edges with get_e/get_v and aggregate per layer pair using links_stat. Then summarize the sign distribution of edge weights (e.g. with plot_e_type_bar or net_par) to determine whether one exposure class (e.g. microbial taxa) is predominantly positive while another (e.g. chemical features) trends negative. Validate against reported counts — for Case 2, 590 microbial taxa linked to 1,983 genes and 245 chemical exposures linked to 1,026 genes — and export statistics such as case2_network_stats.csv. This step operationalizes the cross-layer integration module of MetaNet's end-to-end workflow.

## Related tools

- **MetaNet** (Provides c_net_calculate, c_net_build, get_e/get_v, links_stat, plot_e_type_bar, and net_par for building the network and summarizing cross-layer edge counts and sign distributions) — github.com/Asa12138/MetaNet
- **pcutils** (Companion package loaded alongside MetaNet; supplies test data (otutab) and utilities used in the workflow environment) — github.com/Asa12138/pcutils
- **igraph** (Core dependency underlying MetaNet's network objects, edge extraction, and topological operations)

## Examples

```
cor <- c_net_calculate(t2(data)); net <- c_net_build(cor, r_threshold=0.65, use_p_adj=TRUE); links_stat(net, 'module')
```

## Evaluation signals

- Node counts for cross-layer participation match reported values: 590 microbial taxa linked to 1,983 genes and 245 chemical exposures linked to 1,026 genes
- Sign-pattern invariant: microbial-gene edges are predominantly positive while chemical-gene edges are predominantly negative
- links_stat output accounts for every layer pair with no unclassified edges (intra vs cross-layer e_class sums to total edge count)
- Statistics table (case2_network_stats.csv) is produced and reproducible with a fixed seed
- The built network passed p-value adjustment (use_p_adj) rather than relying on raw p-values — no 'Have not do p-value adjustment!' warning

## Limitations

- Correlation-based network construction may overlook complex, non-linear, or indirect associations
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation
- If p-value adjustment is skipped, c_net_build falls back to raw p-values (README warns: 'Have not do p-value adjustment! use the p.value to build network.')

## Evidence

- [readme] multi-omics workflow: [evidence span withheld — anchor status: external_doc]
- [readme] unadjusted p-values warning: [evidence span withheld — anchor status: external_doc]
