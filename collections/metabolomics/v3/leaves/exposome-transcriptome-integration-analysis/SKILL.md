---
name: exposome-transcriptome-integration-analysis
description: Use when you need to connect exposure-level features (microbial taxa,
  chemicals) to host transcriptomic features via cross-layer correlations and compare
  edge-sign distributions between exposure types.
when_to_use_negative:
- When the goal is single-layer intra-omics analysis (e.g. microbial co-occurrence
  only) rather than cross-layer exposure–gene integration
- When associations are expected to be non-linear or indirect, since correlation-based
  network construction 'may overlook complex, non-linear, or indirect associations'
- When p-value adjustment is not possible or the feature tables are not sample-matched
  across layers
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_1317
tools:
- name: MetaNet
  role: Core R package for cross-layer correlation calculation (c_net_calculate),
    network building (c_net_build/multi_net_build), edge extraction (get_e/get_v),
    and edge-type statistics (links_stat, plot_e_type_bar)
  repo: github.com/Asa12138/MetaNet
- name: pcutils
  role: Companion R package installed via devtools; provides t2 transposition helper
    and example data used alongside MetaNet's correlation workflow
  repo: github.com/Asa12138/pcutils
- name: igraph
  role: Underlying graph library on which MetaNet's network objects and operations
    are built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:cf79f781dff4446c9aa89553b675745b48e6af35d25a0fe2a62293a3d7faa175
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# exposome-transcriptome-integration-analysis

## Summary

Builds an integrative exposome–transcriptome correlation network with MetaNet in R, linking microbial species and chemical exposures to host gene expression, and contrasts the correlation-sign patterns of biological vs chemical exposures.

## When to use

Use when you need to connect exposure-level features (microbial taxa, chemicals) to host transcriptomic features via cross-layer correlations and compare edge-sign distributions between exposure types.

## When NOT to use

- When the goal is single-layer intra-omics analysis (e.g. microbial co-occurrence only) rather than cross-layer exposure–gene integration
- When associations are expected to be non-linear or indirect, since correlation-based network construction 'may overlook complex, non-linear, or indirect associations'
- When p-value adjustment is not possible or the feature tables are not sample-matched across layers

## Inputs

- Gene expression feature table (Case 2: 35,587 transcriptomic genes)
- Microbial abundance feature table (Case 2: 2,955 species)
- Chemical exposure feature table (Case 2: 3,729 chemical features)
- Sample-matched omics matrices with aligned sample IDs
- Filtering parameters: r_threshold, p_threshold, use_p_adj, delete_single

## Outputs

- Integrated exposome–transcriptome igraph/MetaNet network object with cross-layer edges
- Per-layer-pair cross-layer edge counts from get_e/get_v and links_stat
- Edge sign (positive/negative) distribution summaries per exposure type
- Network statistics table (case2_network_stats.csv)

## How to apply

Install MetaNet and pcutils from GitHub via devtools and load the igraph-based environment in R. Load the multi-omics feature tables (Case 2: 35,587 genes, 2,955 microbial species, 3,729 chemical features) and compute pairwise correlations with c_net_calculate, then build the cross-layer network with c_net_build (or multi_net_build) applying r_threshold, p_threshold, use_p_adj (p-value adjustment), and delete_single filtering. Summarize cross-layer edges with get_e/get_v and links_stat, counting associations for each omics layer pair. Characterize edge sign distributions (e.g. plot_e_type_bar or net_par) to verify that microbial exposures are predominantly positively correlated with gene expression while chemical exposures skew negative. Save all network statistics (e.g. to case2_network_stats.csv) for reproducibility.

## Related tools

- **MetaNet** (Core R package for cross-layer correlation calculation (c_net_calculate), network building (c_net_build/multi_net_build), edge extraction (get_e/get_v), and edge-type statistics (links_stat, plot_e_type_bar)) — github.com/Asa12138/MetaNet
- **pcutils** (Companion R package installed via devtools; provides t2 transposition helper and example data used alongside MetaNet's correlation workflow) — github.com/Asa12138/pcutils
- **igraph** (Underlying graph library on which MetaNet's network objects and operations are built)

## Examples

```
multi <- multi_net_build(list(Microbiome=micro, Transcriptome=transc), r_threshold=0.6, p_threshold=0.05, use_p_adj=TRUE)
```

## Evaluation signals

- Reproduce the reported topology: 590 microbial taxa linked to 1,983 genes and 245 chemical exposures linked to 1,026 genes
- Verify the sign-pattern invariant: microbial–gene edges are predominantly positive while chemical–gene edges are predominantly negative
- Confirm cross-layer edge counts per omics layer pair are consistent between get_e/get_v summaries and links_stat output
- Check that p-value adjustment (use_p_adj) was applied before thresholding; building without adjustment triggers the warning 'Have not do p-value adjustment! use the p.value to build network.'
- Confirm all statistics are persisted to case2_network_stats.csv with no single-node vertices left after delete_single filtering

## Limitations

- Correlation-based network construction 'may overlook complex, non-linear, or indirect associations'
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation
- The article section provided contains no data availability or accession information for the two case study datasets, complicating direct reproduction

## Evidence

- [readme] README quick-start build pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] README multi-omics network build: [evidence span withheld — anchor status: not_found]
