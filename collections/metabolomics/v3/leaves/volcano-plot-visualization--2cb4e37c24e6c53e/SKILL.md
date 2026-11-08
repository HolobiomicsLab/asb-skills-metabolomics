---
name: volcano-plot-visualization
description: Use when you have run (or plan to run) run_de() on a metabolites x samples
  abundance matrix with two group labels and want to visualize which metabolites enter
  the Fisher's Exact Test enrichment query set.
when_to_use_negative:
- The query set of metabolites is already known and supplied directly via inputMetabolites
  (e.g. KEGG IDs like C00031, C00022) — no differential analysis, and hence no volcano
  plot, is needed.
- The analysis goal is MetSEA on ranked metabolite lists rather than a two-group comparison;
  the fgsea/MetSEA plot, not the volcano plot, is the relevant visualization.
- Input is a single-group or non-comparative metabolomics profile for which fold change
  between two groups is undefined.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: EnrichMet
  role: Provides run_de() for the two-group differential analysis whose results the
    volcano plot renders, and integrates the volcano plot as an optional ninth plot
    in the enrichmet() output
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Language in which run_de() and the volcano plot generation are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:fb0b7d3f33d086d3df75fc7490dbdc53df87673dd65a1f7726542052030648f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - GetFeatistics
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- volcano-plot-visualisation
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# volcano-plot-visualization

## Summary

Render a volcano plot of the EnrichMet differential analysis output from run_de(), visualizing per-metabolite log2 fold change against significance so that metabolites passing the absolute fold change >= 1 and adjusted p < 0.05 cutoffs can be inspected before pathway enrichment.

## When to use

Use when you have run (or plan to run) run_de() on a metabolites x samples abundance matrix with two group labels and want to visualize which metabolites enter the Fisher's Exact Test enrichment query set.

## When NOT to use

- The query set of metabolites is already known and supplied directly via inputMetabolites (e.g. KEGG IDs like C00031, C00022) — no differential analysis, and hence no volcano plot, is needed.
- The analysis goal is MetSEA on ranked metabolite lists rather than a two-group comparison; the fgsea/MetSEA plot, not the volcano plot, is the relevant visualization.
- Input is a single-group or non-comparative metabolomics profile for which fold change between two groups is undefined.

## Inputs

- Metabolomics abundance matrix (metabolites x samples, e.g. example_data.csv) with control and mutant lung cancer group labels
- run_de() parameters: two group labels (e.g. 'TK-CMV', 'K-CMV'), fc_threshold = 1, pval_threshold = 0.05
- da_results object returned by run_de() (columns met_id, pval or padj, log2fc)

## Outputs

- Volcano plot (S3/S4 plot object) of differential analysis results, as shown in Figure S1
- da_results table of per-metabolite log2 fold change and Benjamini-Hochberg adjusted p-values underlying the plot

## How to apply

Run the differential analysis with run_de() on the metabolomics abundance matrix (e.g., example_data.csv with metabolites as rows, samples as columns) supplying the two group labels (e.g., 'TK-CMV' and 'K-CMV') and the thresholds fc_threshold = 1 and pval_threshold = 0.05. The resulting da_results object carries per-metabolite log2fc, pval/padj columns and significance flags computed with Benjamini-Hochberg adjustment. When differential analysis is run with run_de(), the volcano plot can be included alongside the eight standard enrichmet() plots (pathway, impact, gsea, rbc, network, heatmap, membership, interaction). Use the plot to sanity-check that the significant-metabolite set (absolute fold change >= 1, adjusted p < 0.05) that feeds Fisher's Exact Test pathway enrichment is plausible before committing to the enrichment call. Unlike the other plots, the volcano plot is conditional: it is only produced as part of the end-to-end run_de() -> enrichmet() path, not when the query set is supplied directly via inputMetabolites.

## Related tools

- **EnrichMet** (Provides run_de() for the two-group differential analysis whose results the volcano plot renders, and integrates the volcano plot as an optional ninth plot in the enrichmet() output) — https://github.com/biodatalab/enrichmet.git
- **R** (Language in which run_de() and the volcano plot generation are executed)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05); results_da <- enrichmet(inputMetabolites = NULL, da_results = da_out, ...)
```

## Evaluation signals

- The volcano plot is present in the output only when differential analysis was run via run_de(); it must be absent when enrichmet() is called with a manually supplied inputMetabolites vector.
- Points flagged as significant in the plot should match the filter absolute fold change >= 1 and adjusted p-value < 0.05 used to build the Fisher's Exact Test query set.
- The x-axis reflects log2fc and the y-axis reflects the -log10 of pval/padj columns from the da_results object; the plotted metabolite count must equal nrow(da_results).
- Group labels passed to run_de() (e.g. 'TK-CMV' vs 'K-CMV') must exist as column annotations in the input matrix, otherwise the fold-change computation and plot are invalid.
- Cross-check a few plotted metabolites against the da_results table: direction of fold change and significance status should agree.

## Limitations

- The volcano plot is only available through the run_de() -> enrichmet() end-to-end path; it is not generated for direct-input enrichment runs.
- The plot inherits the run_de() thresholds (fc_threshold = 1, pval_threshold = 0.05); changing the enrichment query criteria requires re-running run_de() to regenerate a consistent plot.
- A volcano plot depicts two-group comparisons only and cannot represent the lipidomics case-study variant unless that data is also framed as control vs mutant contrast.

## Evidence

- [readme] Volcano plot produced only via run_de() path: "If differential analysis is run with `run_de()`, a volcano plot can also
be included."
- [readme] run_de() invocation with thresholds: "da_out <- run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
- [intro] Significance thresholds defining the plotted/metabolite query set: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [other] Volcano plot tied to differential analysis stage in the case study: [evidence span withheld — anchor status: not_found]
