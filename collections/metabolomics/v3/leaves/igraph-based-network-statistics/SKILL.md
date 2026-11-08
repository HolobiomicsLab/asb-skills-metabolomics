---
name: igraph-based-network-statistics
description: Use when a correlation-built omics or multi-omics network (e.g. MetaNet
  'metanet' object) needs quantitative summaries — per-layer-pair link counts, edge
  sign/type distributions, degree rankings — to test hypotheses about network topology.
when_to_use_negative:
- Input is still a raw feature table — network construction (c_net_calculate/c_net_build)
  must run before statistics
- The question concerns network visualization/layout choice rather than quantitative
  edge or sign statistics
- Associations are expected to be non-linear or indirect — the article notes correlation-based
  networks 'may overlook complex, non-linear, or indirect associations'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MetaNet
  role: R package providing get_e/get_v, links_stat, plot_e_type_bar, net_par for
    network statistics on igraph-backed metanet objects
  repo: github.com/Asa12138/MetaNet
- name: igraph
  role: underlying graph library; degree() and graph objects are used directly for
    topology statistics
- name: pcutils
  role: companion package providing data (e.g. otutab) and utilities loaded alongside
    MetaNet in the workflow
  repo: github.com/Asa12138/pcutils
- name: R
  role: execution environment (R >= 4.0) for running MetaNet statistics workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:15352164325f00726f9c6226b87fab5e5a5ffec165ef911913cc5b919588ea08
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# igraph-based-network-statistics

## Summary

Compute and summarize network statistics (cross-layer edge counts, edge sign distributions, degree, links between omics-layer pairs) on igraph-backed MetaNet network objects built from omics correlation matrices.

## When to use

Use when a correlation-built omics or multi-omics network (e.g. MetaNet 'metanet' object) needs quantitative summaries — per-layer-pair link counts, edge sign/type distributions, degree rankings — to test hypotheses about network topology.

## When NOT to use

- Input is still a raw feature table — network construction (c_net_calculate/c_net_build) must run before statistics
- The question concerns network visualization/layout choice rather than quantitative edge or sign statistics
- Associations are expected to be non-linear or indirect — the article notes correlation-based networks 'may overlook complex, non-linear, or indirect associations'

## Inputs

- metanet network object (igraph-backed) from c_net_build / multi_net_build
- correlation matrix from c_net_calculate with r_threshold, p_threshold, use_p_adj, delete_single filters
- multi-omics feature tables (e.g. 35,587 genes, 2,955 microbial species, 3,729 chemical features in Case 2)

## Outputs

- per-layer-pair edge count tables from links_stat (e.g. 590 microbial taxa linked to 1,983 genes; 245 chemical exposures linked to 1,026 genes)
- edge sign/type distribution summaries (plot_e_type_bar, net_par)
- degree rankings and topological statistics saved to CSV (e.g. case2_network_stats.csv)

## How to apply

Build the network first with MetaNet's c_net_calculate (pairwise correlations on the feature table) and c_net_build (apply r_threshold, p_threshold, use_p_adj, delete_single filtering). Extract edges and vertices with get_e/get_v, then use links_stat to tabulate associations per omics-layer pair (e.g. microbial-taxa-to-gene vs chemical-to-gene edges). Summarize edge sign distributions with plot_e_type_bar or net_par to verify sign-pattern hypotheses such as microbial edges being mostly positive and chemical edges mostly negative. Degree can be read directly via the igraph degree() function since MetaNet is 'built upon the widely used igraph package'; counts and statistics should be written to a summary file (e.g. case2_network_stats.csv) for reproducibility.

## Related tools

- **MetaNet** (R package providing get_e/get_v, links_stat, plot_e_type_bar, net_par for network statistics on igraph-backed metanet objects) — github.com/Asa12138/MetaNet
- **igraph** (underlying graph library; degree() and graph objects are used directly for topology statistics)
- **pcutils** (companion package providing data (e.g. otutab) and utilities loaded alongside MetaNet in the workflow) — github.com/Asa12138/pcutils
- **R** (execution environment (R >= 4.0) for running MetaNet statistics workflows)

## Examples

```
library(MetaNet); net <- c_net_build(c_net_calculate(totu), r_threshold = 0.65, p_threshold = 0.05); links_stat(net, "e_class")
```

## Evaluation signals

- Cross-layer counts reproduce the published Case 2 numbers: 590 microbial taxa linked to 1,983 genes and 245 chemical exposures linked to 1,026 genes
- Edge sign distributions confirm microbial exposures are predominantly positively correlated with gene expression while chemical exposures show a bias toward negative associations
- links_stat output partitions all edges into intra-/inter-omic layer pairs with no unassigned edges after delete_single filtering
- Degree ranking from igraph degree() is sortable and consistent with the network's vertex set (e.g. head of sorted degrees returns hub features)
- Statistics are saved to a standardized output file (case2_network_stats.csv) as the workflow requires

## Limitations

- Correlation-based network statistics may overlook complex, non-linear, or indirect associations
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation
- If p-value adjustment was not applied, MetaNet warns 'Have not do p-value adjustment! use the p.value to build network.' — sign/count statistics then reflect unadjusted significance
- No data availability or accession information was provided for the two case study datasets in the article section

## Evidence

- [readme] links_stat usage: [evidence span withheld — anchor status: external_doc]
- [readme] p-value adjustment warning: [evidence span withheld — anchor status: external_doc]
