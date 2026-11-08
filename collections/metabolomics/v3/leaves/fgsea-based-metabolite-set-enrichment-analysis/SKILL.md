---
name: fgsea-based-metabolite-set-enrichment-analysis
description: Use when you have metabolomics summary statistics (log2FC and p-values
  per metabolite) and want GSEA-style enrichment over KEGG pathway sets, e.g. reproducing
  control vs mutant lung cancer MetSEA results.
when_to_use_negative:
- Input is only an unranked list of significant metabolites without log2FC and p-value
  statistics — use the Fisher's Exact Test over-representation module instead, since
  MetSEA requires a ranked list.
- The goal is topology-based pathway impact (betweenness centrality / impact score)
  or the STITCH interaction network rather than ranked-set enrichment.
- No pathway annotations are available and KEGG REST access is not possible — the
  package fetches annotations at runtime and does not ship database dumps.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: EnrichMet
  role: R package providing the MetSEA module that wraps fgsea on a ranked metabolite
    list against GMT-formatted KEGG pathway annotations
  repo: https://github.com/biodatalab/enrichmet.git
- name: fgsea
  role: Underlying fast GSEA engine called by the MetSEA module to compute NES and
    adjusted p-values on the ranked metabolite list
- name: KEGG
  role: Source of curated pathway-metabolite association data converted into GMT format
    for MetSEA
- name: R
  role: Language in which the MetSEA/fgsea workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:4388b9ffd84500b3e1cde8d2cffb9693dc1d48eb087dbe1c968e2db275170e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - fgsea
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fgsea-based-metabolite-set-enrichment-analysis

## Summary

MetSEA in EnrichMet runs metabolite set enrichment analysis with fgsea on metabolites ranked by log2 fold change and p-value, against KEGG pathway annotations converted to GMT format, returning NES and adjusted p-values per pathway.

## When to use

Use when you have metabolomics summary statistics (log2FC and p-values per metabolite) and want GSEA-style enrichment over KEGG pathway sets, e.g. reproducing control vs mutant lung cancer MetSEA results.

## When NOT to use

- Input is only an unranked list of significant metabolites without log2FC and p-value statistics — use the Fisher's Exact Test over-representation module instead, since MetSEA requires a ranked list.
- The goal is topology-based pathway impact (betweenness centrality / impact score) or the STITCH interaction network rather than ranked-set enrichment.
- No pathway annotations are available and KEGG REST access is not possible — the package fetches annotations at runtime and does not ship database dumps.

## Inputs

- Ranked metabolite summary statistics with columns met_id, pval (or padj), and log2fc (e.g. output of enrichmet's run_de on the control vs mutant lung cancer matrix)
- KEGG pathway-to-metabolite association table in GMT format (e.g. PathwayVsMetabolites fetched for organism 'hsa')

## Outputs

- MetSEA results table (data.frame) with per-pathway pval, padj, and NES columns
- Significant pathway list filtered at adjusted p-value < 0.05
- MetSEA plot (gsea_plot / dot plot of -log10(adjusted p-values) for top enriched pathways)

## How to apply

First convert the curated KEGG pathway-to-metabolite associations into Gene Matrix Transposed (GMT) format so they are compatible with fgsea (via fetch_kegg_pathway_metabolites, e.g. organism 'hsa'). Rank metabolites from the differential analysis (e.g. run_de with fc_threshold = 1, pval_threshold = 0.05) by log2 fold change and p-value — MetSEA requires summary statistics with met_id, pval/padj, and log2fc columns rather than a bare metabolite list. Call the MetSEA module (enrichmet() with analysis_type including 'gsea', or fgsea directly on the ranked list plus the GMT file). A positive or negative NES accompanied by adjusted p-value < 0.05 marks a significantly enriched pathway; filter the fgsea output table to those pathways, keeping NES and adjusted-p columns. Visualize top hits as a dot plot of -log10(adjusted p-values); in the lung cancer case study the expected recovered categories were metabolic reprogramming and energy production, nutrient uptake and homeostasis (Figure 2C).

## Related tools

- **EnrichMet** (R package providing the MetSEA module that wraps fgsea on a ranked metabolite list against GMT-formatted KEGG pathway annotations) — https://github.com/biodatalab/enrichmet.git
- **fgsea** (Underlying fast GSEA engine called by the MetSEA module to compute NES and adjusted p-values on the ranked metabolite list)
- **KEGG** (Source of curated pathway-metabolite association data converted into GMT format for MetSEA)
- **R** (Language in which the MetSEA/fgsea workflow is executed)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, analysis_type = c("enrichment", "gsea"))
```

## Evaluation signals

- The MetSEA results table contains NES and adjusted-p (padj) columns for each tested pathway.
- Significant pathways are defined by adjusted p-value < 0.05, matching the paper's MetSEA significance criterion.
- The sign of NES (positive or negative) is consistent with the direction of the log2 fold-change ranking of metabolites.
- On the control vs mutant lung cancer case study, the top enriched pathways group into the expected Figure 2C categories: metabolic reprogramming and energy production, nutrient uptake and homeostasis.
- Pathway annotations must be in GMT format (Gene Matrix Transposed) for fgsea compatibility — verify the annotation conversion before running.

## Limitations

- MetSEA requires ranked statistics (log2 fold change and p-values); it cannot be applied to a plain metabolite identifier list.
- With small metabolite sets or sparse statistics, adjusted p-values may fail to reach significance even for low raw p-values (README example shows top GSEA pathways with padj = 0.316 despite pval = 0.0014).
- KEGG/Reactome annotations are fetched at runtime via public APIs and cached locally with BiocFileCache; the package does not redistribute third-party database dumps, so network access is needed.
- No changelog is documented for the package.

## Evidence

- [intro] Pathway annotations are converted into a Gene Matrix Transposed (GMT) format to ensure compatibility with fgsea: "Pathway annotations are converted into a Gene Matrix Transposed (GMT) format to ensure compatibility with fgsea"
- [intro] MetSEA significance criterion: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [abstract] MetSEA module scope: "EnrichMet modules support over-representation analysis of pathways, metabolite set enrichment analysis (MetSEA), and network-based pathway analysis"
- [readme] MetSEA implementation via fgsea: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] MetSEA input requirements: [evidence span withheld — anchor status: external_doc]
