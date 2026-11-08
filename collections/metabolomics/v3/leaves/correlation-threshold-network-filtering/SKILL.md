---
name: correlation-threshold-network-filtering
description: Use when constructing co-occurrence or cross-omics association networks
  from feature abundance tables (e.
when_to_use_negative:
- Input is raw reads or per-sample metadata rather than a processed feature abundance
  table matched across samples (c_net_build requires matched sample columns, 'All
  samples matched.')
- Analysis targets non-linear or indirect associations — the article notes correlation-based
  networks 'may overlook complex, non-linear, or indirect associations'
- Dataset requires a similarity metric or null model other than simple correlation/significance
  filtering, where standardized threshold selection is itself contested in the literature
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: MetaNet
  role: Primary R package providing c_net_calculate, c_net_build, c_net_filter, and
    multi_net_build for threshold-filtered correlation network construction
  repo: github.com/Asa12138/MetaNet
- name: igraph
  role: Underlying graph library on which MetaNet's network objects and edge filtering
    operations are built
- name: pcutils
  role: Companion package supplying example otutab data and t2() transposition helper
    used before correlation calculation
  repo: github.com/Asa12138/pcutils
- name: R
  role: Runtime environment (R >= 4.0) in which the MetaNet filtering pipeline is
    executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:7f7470de9d87c47e8b63f370a4849a520f0c71e49456fa635a40044628eb81ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# correlation-threshold-network-filtering

## Summary

Builds sparse correlation networks from high-dimensional omics feature tables by computing pairwise correlations and retaining only edges passing correlation (r_threshold) and statistical significance (p_threshold, p-value adjustment) filters, using MetaNet's c_net_calculate/c_net_build pipeline.

## When to use

Use when constructing co-occurrence or cross-omics association networks from feature abundance tables (e.g., microbial ASV/species counts, metabolite or transcript counts) where pairwise correlation edges must be filtered by a correlation threshold and p-value significance before downstream topology or sign-distribution analysis.

## When NOT to use

- Input is raw reads or per-sample metadata rather than a processed feature abundance table matched across samples (c_net_build requires matched sample columns, 'All samples matched.')
- Analysis targets non-linear or indirect associations — the article notes correlation-based networks 'may overlook complex, non-linear, or indirect associations'
- Dataset requires a similarity metric or null model other than simple correlation/significance filtering, where standardized threshold selection is itself contested in the literature

## Inputs

- Feature abundance table (samples x features), e.g., OTU/ASV count table from pcutils 'otutab' or transposed species table
- Named list of omics layer matrices for multi-omics network construction (e.g., Microbiome, Metabolome, Transcriptome count matrices)
- Threshold parameters: r_threshold, p_threshold, use_p_adj flag, delete_single flag

## Outputs

- A MetaNet/igraph network object containing only edges passing the correlation and significance thresholds
- Correlation coefficient and p-value matrix from c_net_calculate
- Filtered sub-networks from c_net_filter for downstream edge-sign or links_stat analysis

## How to apply

First compute all pairwise correlations between features (or between layers of a multi-omics dataset) with c_net_calculate, optionally transposing samples-by-features tables with t2(). Then build the network with c_net_build, setting r_threshold (e.g., 0.6–0.65 in the README quick-start) and p_threshold, and enable p-value adjustment via use_p_adj since the default run warns 'Have not do p-value adjustment! use the p.value to build network.' Use delete_single to drop isolated single-edge vertices. For multi-omics input, use multi_net_build over a named list of omics layers (e.g., list(Microbiome=..., Metabolome=..., Transcriptome=...)) with the same r_threshold logic; MetaNet is optimized so this remains tractable for >10,000 features. Decision point: threshold choice is typically subjective and can bias network topology, so sensitivity-check by varying r_threshold and record whether p-values were adjusted. Filtered networks can then be subset with c_net_filter (e.g., restrict to specific v_group layers or e_class=='intra') before edge-sign or links_stat summaries.

## Related tools

- **MetaNet** (Primary R package providing c_net_calculate, c_net_build, c_net_filter, and multi_net_build for threshold-filtered correlation network construction) — github.com/Asa12138/MetaNet
- **igraph** (Underlying graph library on which MetaNet's network objects and edge filtering operations are built)
- **pcutils** (Companion package supplying example otutab data and t2() transposition helper used before correlation calculation) — github.com/Asa12138/pcutils
- **R** (Runtime environment (R >= 4.0) in which the MetaNet filtering pipeline is executed)

## Examples

```
library(pcutils); library(MetaNet); cor <- c_net_calculate(t2(otutab[1:70, ])); net <- c_net_build(cor, r_threshold = 0.65, p_threshold = 0.05, use_p_adj = TRUE)
```

## Evaluation signals

- c_net_build must emit or you must resolve the p-value adjustment warning; a network built on unadjusted p-values is incomplete ('Have not do p-value adjustment!')
- Edge counts in the built network should change monotonically with r_threshold; verify by re-running c_net_build at adjacent thresholds
- No isolated vertices remain when delete_single is enabled (each retained node has >=1 passing edge)
- For the Case 2 exposome-transcriptome application, filtered cross-layer edges must reproduce the reported topology (590 microbial taxa linked to 1,983 genes; 245 chemical exposures linked to 1,026 genes)
- Vertex and edge group attributes (v_group, e_class) must be set correctly so c_net_filter subsetting by layer works as intended

## Limitations

- Correlation-based network construction can overlook complex, non-linear, or indirect associations
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation
- Without explicit use_p_adj, c_net_build falls back on unadjusted p-values, which can inflate false-positive edges at high feature counts
- Computational cost of full pairwise correlation can be a bottleneck for high-dimensional datasets in tools without MetaNet's vectorized optimization (mitigated here for >10,000 features)

## Evidence

- [abstract] supports highly efficient correlation-based network construction, scalable to datasets with over 10,000 features: "supports highly efficient correlation-based network construction, scalable to datasets with over 10,000 features"
- [methods] devtools install GitHub: "devtools::install_github("Asa12138/MetaNet")"
- [readme] README quick-start threshold example: "net <- c_net_build(cor, r_threshold = 0.65)
#> Have not do p-value adjustment! use the p.value to build network."
- [readme] README multi-omics build call: "multi1 <- multi_net_build(list(
  Microbiome = micro,
  Metabolome = metab,
  Transcriptome = transc
), r_threshold = 0.6)"
- [intro] Thresholding subjectivity caveat: [evidence span withheld — anchor status: not_found]
- [discussion] Non-linearity limitation: "may overlook complex, non-linear, or indirect associations"
