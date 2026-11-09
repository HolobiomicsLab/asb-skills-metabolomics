---
name: benjamini-hochberg-fdr-edge-significance-testing
description: Use when constructing a multi-omics co-occurrence network from paired
  feature tables (e.
when_to_use_negative:
- If the input is already a pre-computed network edge list (e.g., from Cytoscape or
  Gephi) with no need for statistical significance testing on edges.
- If the dataset is small (fewer than ~50 features) and the user prefers uncorrected
  p-value thresholds; BH-FDR is overly conservative for very small numbers of tests.
- If the goal is not to build a new network but to visualize an existing `metanet`
  object — use `c_net_plot` directly instead.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MetaNet
  role: Provides `c_net_build` with `p_adjust_method = "BH"` for applying Benjamini-Hochberg
    FDR correction to correlation p-values during network edge selection.
  repo: https://github.com/Asa12138/MetaNet
- name: pcutils
  role: Contains helper functions (`t2`, `otutab`) used in the workflow to transpose
    feature tables and calculate abundance-based attributes for annotation.
  repo: https://github.com/Asa12138/pcutils
- name: igraph
  role: Core dependency for the `metanet` network object; provides graph data structure,
    degree computation, and neighbor queries underlying the annotated network.
  repo: https://github.com/igraph/igraph
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:12c45fd78747fcb1057b589c62793da8fa4395e07ba72d2b288db4842a7efed6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - R
  - ggraph
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# benjamini-hochberg-fdr-edge-significance-testing

## Summary

A workflow for building and annotating a multi-omics co-occurrence network (microbiome, metabolome, transcriptome) using MetaNet, followed by edge-significance testing with Benjamini-Hochberg FDR correction. This skill uses the `c_net_build` function's `p_adjust_method` parameter to apply BH-FDR on correlation p-values before thresholding, ensuring reproducible and statistically controlled edge inference.

## When to use

Use when constructing a multi-omics co-occurrence network from paired feature tables (e.g., species abundances, metabolite peak areas, gene counts) where pairwise correlation p-values must be corrected for multiple testing to control false discovery rate among inferred edges, and where downstream module/topology analysis requires statistically defensible edge selection.

## When NOT to use

- If the input is already a pre-computed network edge list (e.g., from Cytoscape or Gephi) with no need for statistical significance testing on edges.
- If the dataset is small (fewer than ~50 features) and the user prefers uncorrected p-value thresholds; BH-FDR is overly conservative for very small numbers of tests.
- If the goal is not to build a new network but to visualize an existing `metanet` object — use `c_net_plot` directly instead.

## Inputs

- Multi-omics feature tables as a named list (e.g., `list(Microbiome = micro, Metabolome = metab, Transcriptome = transc)`), with rows=samples and columns=features
- Metadata annotation tables (e.g., `micro_g`, `metab_g`, `transc_g`) for vertex attributes such as `Phylum`, `kingdom`, or `type`
- A correlation matrix object returned by `c_net_calculate` if building the network manually

## Outputs

- A `metanet` network object (igraph-based) with significant edges retained after BH-FDR correction
- A network plot via `c_net_plot()` showing annotated nodes and significant edges
- A `links_stat` table or `c_net_skeleton` object summarizing edge origins/targets across modules or conditions

## How to apply

1. Load the `MetaNet` and `pcutils` R packages (install via `install.packages("MetaNet")` or `devtools::install_github("Asa12138/MetaNet")`). 2. Prepare the multi-omics tables as a named list (e.g., `list(Microbiome = micro, Metabolome = metab, Transcriptome = transc)`), with samples in rows and features in columns. 3. Build the network using `c_net_build(cor, r_threshold = 0.6, p_threshold = 0.05, use_p_adj = TRUE, p_adjust_method = "BH")`; this computes Spearman or Pearson correlations, then applies Benjamini-Hochberg FDR correction to the p-values before applying the r_threshold and p_threshold cutoffs to retain significant edges. 4. Annotate vertex attributes (e.g., class, size, color) using `c_net_set` to map metadata columns such as `Phylum`, `kingdom`, or `type` onto the network nodes. 5. The rationale is that uncorrected p-values inflate false positive edges in high-dimensional multi-omics datasets; applying BH-FDR controls the expected proportion of false discoveries among all edges. Decision point: if the network appears too dense or sparse, adjust `r_threshold` (e.g., from 0.6 to 0.65) while keeping `use_p_adj = TRUE` and `p_adjust_method = "BH"`.

## Related tools

- **MetaNet** (Provides `c_net_build` with `p_adjust_method = "BH"` for applying Benjamini-Hochberg FDR correction to correlation p-values during network edge selection.) — https://github.com/Asa12138/MetaNet
- **pcutils** (Contains helper functions (`t2`, `otutab`) used in the workflow to transpose feature tables and calculate abundance-based attributes for annotation.) — https://github.com/Asa12138/pcutils
- **igraph** (Core dependency for the `metanet` network object; provides graph data structure, degree computation, and neighbor queries underlying the annotated network.) — https://github.com/igraph/igraph

## Examples

```
library(MetaNet); library(pcutils); cor <- c_net_calculate(t2(otutab[1:70, ])); net <- c_net_build(cor, r_threshold = 0.6, p_threshold = 0.05, use_p_adj = TRUE, p_adjust_method = "BH"); c_net_plot(net)
```

## Evaluation signals

- Verify that the resulting network's edge count is substantially lower than the number of edges obtained with `use_p_adj = FALSE` at the same `r_threshold`, reflecting the conservativeness of BH-FDR.
- Check that all retained edges have `p_adj <= p_threshold` (e.g., 0.05) by inspecting the edge attributes in the `metanet` object.
- Confirm that the network object is a valid `metanet`/igraph structure by calling `is.metanet()` and checking `E(net)$r` and `E(net)$p_adj` attributes exist for all edges.
- Compare the resulting network against the one built from `c_net_calculate` + `c_net_build` with `p_adjust_method = "BH"` on the same data — edge sets should match exactly (reproducibility check).
- Ensure that the BH-FDR step produces a warning or message only when `p_adjust_method` is explicitly specified, distinguishing it from the default uncorrected path (e.g., the 'Have not do p-value adjustment!' warning should NOT appear).

## Limitations

- The default path in `c_net_build` (when `p_adjust_method` is not specified) does not perform p-value adjustment and instead uses raw p-values, so users must explicitly set `p_adjust_method = "BH"` to invoke this skill's statistical correction.
- MetaNet is built on igraph and is compatible only with R ≥ 4.0; it does not support running in Python or other languages directly.
- While BH-FDR controls the expected proportion of false discoveries, it does not guarantee that every retained edge is biologically meaningful; further validation (e.g., stability analysis or cross-validation) is recommended.

## Evidence

- [readme] metaNet enables fast and scalable correlation-based network construction: "MetaNet enables fast and scalable correlation-based network construction for datasets with more than 10,000 features"
- [readme] network construction with p-value adjustment: [evidence span withheld — anchor status: external_doc]
- [readme] correlation calculation and network building: [evidence span withheld — anchor status: external_doc]
- [readme] edge filtering by significant correlation: [evidence span withheld — anchor status: not_found]
- [readme] annotation of vertex attributes: [evidence span withheld — anchor status: external_doc]
- [readme] network construction function: "network <- c_net_build(cor, r_threshold = 0.65)
#> Have not do p-value adjustment! use the p.value to build network."
