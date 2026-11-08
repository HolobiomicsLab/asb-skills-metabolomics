---
name: kegg-pathway-enrichment-interpretation
description: Use when a differential metabolomics experiment (e.g., control vs mutant
  lung cancer) yields a metabolite list or ranked statistics and you must identify
  significantly enriched KEGG pathways and prioritize them by biological impact.
when_to_use_negative:
- When the input is already a generic gene/feature set without metabolite KEGG IDs
  or metabolite summary statistics (met_id, pval/padj, log2fc) — MetSEA ranking and
  the KEGG mapping cannot be constructed
- When the goal is lipid ontology interpretation rather than KEGG pathway enrichment
  — the lipidomics extension fetches LION associations at runtime and follows a different
  mapping path
- When no topology/impact interpretation is wanted and only a raw unfiltered fgsea
  run is needed — the adjusted p < 0.05 filter and impact-score combination would
  discard borderline pathways
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3822
tools:
- name: EnrichMet
  role: Primary R package; its modular components run Fisher's Exact Test ORA, fgsea-based
    MetSEA, and betweenness-centrality pathway impact analysis in a single enrichmet()
    call
  repo: https://github.com/biodatalab/enrichmet.git
- name: fgsea
  role: 'Backend engine for MetSEA: consumes GMT-formatted KEGG pathway annotations
    and the log2 fold change / p-value ranked metabolite list to produce NES and adjusted
    p-values'
- name: KEGG
  role: Source of curated pathway-metabolite association data converted into GMT format
    and fetched at runtime (e.g., fetch_kegg_pathway_metabolites(organism = 'hsa'))
- name: STITCH
  role: Source of chemical-chemical relationships (filtered at combined score >= 100)
    used to build the metabolite-metabolite network for pathway impact scoring
- name: R
  role: Runtime environment; EnrichMet is installed via BiocManager and returns S3/S4
    data.frame and plot objects
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:336a631399b925063d0a71e5e3f6217c776030dc044aca34e45a392d42c81ad2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - KEGGREST
  - MetENP
  - R
  - STITCH
  - fgsea
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3822
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- kegg-pathway-enrichment-scoring
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kegg-pathway-enrichment-interpretation

## Summary

Interpret KEGG-based metabolite pathway enrichment results from EnrichMet by reading ORA (Fisher's Exact Test) tables and fgsea-based MetSEA tables (NES, adjusted p < 0.05) and ranking pathways by both statistical enrichment and topology-based pathway impact.

## When to use

Use when a differential metabolomics experiment (e.g., control vs mutant lung cancer) yields a metabolite list or ranked statistics and you must identify significantly enriched KEGG pathways and prioritize them by biological impact.

## When NOT to use

- When the input is already a generic gene/feature set without metabolite KEGG IDs or metabolite summary statistics (met_id, pval/padj, log2fc) — MetSEA ranking and the KEGG mapping cannot be constructed
- When the goal is lipid ontology interpretation rather than KEGG pathway enrichment — the lipidomics extension fetches LION associations at runtime and follows a different mapping path
- When no topology/impact interpretation is wanted and only a raw unfiltered fgsea run is needed — the adjusted p < 0.05 filter and impact-score combination would discard borderline pathways

## Inputs

- Character vector of significant metabolites as KEGG IDs (e.g., C00031, C00022) or names, with input_type = 'kegg' | 'name' | 'mixed'
- Summary statistics table with columns met_id, pval or padj, and log2fc (for MetSEA and measured-metabolite background)
- KEGG pathway-to-metabolite mapping (Pathway/Metabolites table, e.g., organism 'hsa')
- STITCH chemical-chemical interaction list for the pathway impact/network module
- Output of run_de() on a metabolomics matrix (metabolites as rows, samples as columns) for end-to-end analysis

## Outputs

- Pathway enrichment table (Pathway, P_value, Adjusted_P_value, Enrichment_Ratio) filtered at adjusted p < 0.05
- MetSEA/fgsea results table (pathway, pval, padj, NES) for significantly enriched pathways
- Metabolite centrality table (Display_Name, RBC_Metabolite) and pathway impact scores
- Dot plots of -log10(adjusted p-values) reproducing Figure 2B/2C-style pathway impact and enrichment visualizations

## How to apply

Start from significant metabolites selected at absolute fold change >= 1 and adjusted p < 0.05, or from a ranked list ordered by log2 fold change and p-value. Run EnrichMet's ORA module (Fisher's Exact Test on a 2x2 contingency table with Benjamini-Hochberg adjustment) and its MetSEA module, which converts KEGG pathway annotations to GMT format and calls fgsea on the ranked metabolites. Filter the enrichment tables to pathways with adjusted p-value < 0.05, keeping NES and adjusted-p fields, where positive or negative NES indicates the enrichment direction. For prioritization beyond statistics, build the metabolite-metabolite interaction network from STITCH chemical-chemical relationships filtered at combined score >= 100, compute betweenness-centrality-derived pathway impact scores, and combine them with -log10(adjusted p-values) in a dot-plot (Figure 2B/2C style). Interpretation is correct when top statistical hits (e.g., pyrimidine metabolism) and structurally impactful hits (e.g., caffeine metabolism) both surface, and recovered categories match the paper's grouping of metabolic reprogramming and energy production/nutrient uptake/homeostasis.

## Related tools

- **EnrichMet** (Primary R package; its modular components run Fisher's Exact Test ORA, fgsea-based MetSEA, and betweenness-centrality pathway impact analysis in a single enrichmet() call) — https://github.com/biodatalab/enrichmet.git
- **fgsea** (Backend engine for MetSEA: consumes GMT-formatted KEGG pathway annotations and the log2 fold change / p-value ranked metabolite list to produce NES and adjusted p-values)
- **KEGG** (Source of curated pathway-metabolite association data converted into GMT format and fetched at runtime (e.g., fetch_kegg_pathway_metabolites(organism = 'hsa')))
- **STITCH** (Source of chemical-chemical relationships (filtered at combined score >= 100) used to build the metabolite-metabolite network for pathway impact scoring)
- **R** (Runtime environment; EnrichMet is installed via BiocManager and returns S3/S4 data.frame and plot objects)

## Examples

```
results <- enrichmet(inputMetabolites=inputMetabolites, PathwayVsMetabolites=PathwayVsMetabolites, example_data=example_summary, kegg_lookup=kegg_lookup, reactome_df=reactome_df, analysis_type=c("enrichment","gsea","centrality","network"))
```

## Evaluation signals

- The MetSEA enrichment table contains NES and adjusted-p columns, and significant pathways are those with adjusted p-value < 0.05 as in the paper's MetSEA criterion
- Recovered enrichment categories include metabolic reprogramming and energy production/nutrient uptake/homeostasis, matching the Figure 2C grouping
- Both pyrimidine metabolism (top adjusted p-value) and caffeine metabolism (strong structural impact) appear among the high-impact significant pathways, matching the reported Figure 2B pattern
- ORA columns include P_value, Adjusted_P_value, and Enrichment_Ratio (e.g., glycolysis/gluconeogenesis adjusted p ~1.36e-16, ratio ~85.8 in the README example)
- Enrichment on the 94-metabolite representative dataset completes in approximately 3 seconds (vs ~28 s for MetaboAnalystR and ~14 min for FELLA)

## Limitations

- Fisher's Exact Test can be biased by database compounds never observed in the experiment; EnrichMet mitigates this by restricting the test to a measured background (backgroundMetabolites from extract_measured_kegg_ids)
- STITCH interaction quality depends on the combined score >= 100 filter; weaker interactions are excluded to ensure biological relevance
- Pathway and compound annotations are fetched at runtime from KEGG REST and Reactome mappings and cached with BiocFileCache, so the workflow requires network access on first run and does not redistribute database dumps

## Evidence

- [intro] MetSEA significance criterion: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [intro] GMT conversion for fgsea: "Pathway annotations are converted into a Gene Matrix Transposed (GMT) format to ensure compatibility with fgsea"
- [intro] Pathway impact quantification: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [intro] STITCH filter: "STITCH interactions were filtered based on combined score (≥100)"
- [readme] README MetSEA engine: "performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**. The `enrichmet()` function produces three tables (S3 `data.frame` objects)"
- [readme] Measured-background correction: "Fisher’s exact test can be restricted to metabolites that were measured
(testable) in the experiment, reducing bias from database compounds that
were never observed"
