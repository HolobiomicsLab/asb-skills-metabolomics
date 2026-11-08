---
name: metabolomics-pathway-enrichment-execution
description: Use when a metabolomics study yields a list of significant metabolites
  (KEGG IDs or names) and the goal is fast, local pathway enrichment and network analysis,
  especially when speed or reproducibility matters.
when_to_use_negative:
- Input is already a precomputed enrichment result table or a pathway-impact score
  table rather than a metabolite query set — EnrichMet expects raw significant-metabolite
  input, not downstream analysis artifacts.
- The target organism or pathway source is not covered by KEGG/Reactome/LION and no
  user-supplied Pathway/Metabolites mapping table exists — the package does not redistribute
  third-party database dumps and requires runtime APIs or a supplied equivalent.
- The analysis goal is lipid-specific ontology enrichment outside the scope of the
  KEGG pathway map, unless the LION lipid ontology is fetched at runtime via fetch_lion_lipid_ontology()
  — do not assume built-in lipid pathways.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3391
tools:
- name: EnrichMet
  role: Primary R package that executes the integrated enrichment workflow (Fisher's
    Exact Test ORA, MetSEA via fgsea, betweenness centrality network analysis) in
    a single function call
  repo: https://github.com/biodatalab/enrichmet.git
- name: MetaboAnalystR
  role: Alternative R-based pathway enrichment tool used for runtime benchmarking
    comparison against EnrichMet (~28 seconds vs ~3 seconds on the 94-metabolite dataset)
- name: FELLA
  role: Network-based pathway analysis tool used for runtime benchmarking comparison
    against EnrichMet (~14 minutes on the same 94-metabolite dataset)
- name: fgsea
  role: Backend used inside EnrichMet to perform MetSEA (Metabolite Set Enrichment
    Analysis) on ranked metabolite lists with GMT-formatted pathway annotations
- name: KEGG
  role: Source of curated pathway-to-metabolite annotations fetched at runtime via
    KEGG REST API and cached with BiocFileCache
- name: STITCH
  role: Source of chemical-chemical relationships used to build the metabolite-metabolite
    interaction network, filtered at combined score >= 100
- name: LION
  role: Lipid ontology database fetched at runtime to extend the enrichment framework
    to lipidomics data
- name: Reactome
  role: Source of reaction mappings used for the metabolite interaction network analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:241370df1d4db96ad54641378798db3e1e236e6e207ebbe36a5b000eab85ef53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - FELLA
  - KEGG
  - LION
  - MetaboAnalystR
  - R
  - Reactome
  - STITCH
  - fgsea
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# metabolomics-pathway-enrichment-execution

## Summary

Execute integrated metabolomics pathway enrichment via the EnrichMet R package, combining Fisher's Exact Test over-representation analysis, fgsea-based MetSEA on ranked metabolite lists, and topology-based network analysis using STITCH-derived interaction networks in a single function call.

## When to use

Use when a metabolomics study yields a list of significant metabolites (KEGG IDs or names) and the goal is fast, local pathway enrichment and network analysis, especially when speed or reproducibility matters.

## When NOT to use

- Input is already a precomputed enrichment result table or a pathway-impact score table rather than a metabolite query set — EnrichMet expects raw significant-metabolite input, not downstream analysis artifacts.
- The target organism or pathway source is not covered by KEGG/Reactome/LION and no user-supplied Pathway/Metabolites mapping table exists — the package does not redistribute third-party database dumps and requires runtime APIs or a supplied equivalent.
- The analysis goal is lipid-specific ontology enrichment outside the scope of the KEGG pathway map, unless the LION lipid ontology is fetched at runtime via fetch_lion_lipid_ontology() — do not assume built-in lipid pathways.

## Inputs

- Query set of significant metabolites as character vector of KEGG IDs and/or names (input_type: "kegg" (default), "name", or "mixed")
- Pathway-to-metabolite mapping table with Pathway/Metabolites columns (fetched via fetch_kegg_pathway_metabolites or user-supplied)
- Summary statistics table with columns met_id, pval or padj, and log2fc (required for MetSEA or measured-background construction)
- KEGG compound ID-to-name lookup (fetch_kegg_compound_lookup, needed for name/mixed input)
- Reactome reaction mappings (fetch_kegg_reactome, needed for interaction network analysis)
- Measured-metabolite background list (e.g. extract_measured_kegg_ids) for background-corrected Fisher's test
- Raw metabolomics matrix (metabolites as rows, samples as columns) if starting from run_de() differential analysis

## Outputs

- Pathway enrichment results table (S3 data.frame with Pathway, P_value, Adjusted_P_value, Enrichment_Ratio columns)
- MetSEA results table (fgsea output with pathway, pval, padj, NES columns)
- Metabolite centrality table (Display_Name, RBC_Metabolite columns)
- Pathway enrichment dot plot of -log10(adjusted p-values)
- Pathway impact plot (significance vs. topology-based impact score)
- MetSEA enrichment plot
- Relative Betweenness Centrality (RBC) plot
- Pathway-metabolite network graph
- Pathway heatmap
- Pathway membership plot (ComplexHeatmap)
- Reactome interaction network plot
- Optional volcano plot if differential analysis is run with run_de()

## How to apply

Prepare a query set of significant metabolites, filtered at absolute fold change >= 1 and adjusted p-value < 0.05, as KEGG IDs, names, or a mixed input via `input_type` ("kegg", "name", "mixed"). Fetch pathway-metabolite annotations at runtime with `fetch_kegg_pathway_metabolites(organism = "hsa")` and KEGG compound lookups, caching with BiocFileCache; for network analysis also fetch Reactome reaction mappings with `fetch_kegg_reactome()`. Call `enrichmet()` with the query set, the PathwayVsMetabolites table, summary statistics (columns `met_id`, `pval`/`padj`, `log2fc`), and the Reactome mappings; the function runs Fisher's Exact Test over-representation analysis on a 2x2 contingency table with Benjamini-Hochberg adjustment, MetSEA via fgsea on GMT-formatted pathway annotations using ranked lists (log2 fold change and p-values), and computes relative betweenness centrality from STITCH chemical-chemical interaction networks filtered at combined score >= 100. Optionally restrict Fisher's test to a measured-metabolite background via `backgroundMetabolites` to reduce bias from unobserved database compounds. The single call returns three tables (pathway enrichment results, MetSEA results, metabolite centrality) and eight plots, and can be benchmarked for runtime: on a representative 94-metabolite dataset EnrichMet completes in approximately 3 seconds, compared with ~28 seconds for MetaboAnalystR and ~14 minutes for FELLA.

## Related tools

- **EnrichMet** (Primary R package that executes the integrated enrichment workflow (Fisher's Exact Test ORA, MetSEA via fgsea, betweenness centrality network analysis) in a single function call) — https://github.com/biodatalab/enrichmet.git
- **MetaboAnalystR** (Alternative R-based pathway enrichment tool used for runtime benchmarking comparison against EnrichMet (~28 seconds vs ~3 seconds on the 94-metabolite dataset))
- **FELLA** (Network-based pathway analysis tool used for runtime benchmarking comparison against EnrichMet (~14 minutes on the same 94-metabolite dataset))
- **fgsea** (Backend used inside EnrichMet to perform MetSEA (Metabolite Set Enrichment Analysis) on ranked metabolite lists with GMT-formatted pathway annotations)
- **KEGG** (Source of curated pathway-to-metabolite annotations fetched at runtime via KEGG REST API and cached with BiocFileCache)
- **STITCH** (Source of chemical-chemical relationships used to build the metabolite-metabolite interaction network, filtered at combined score >= 100)
- **LION** (Lipid ontology database fetched at runtime to extend the enrichment framework to lipidomics data)
- **Reactome** (Source of reaction mappings used for the metabolite interaction network analysis)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, input_type = "kegg")
```

## Evaluation signals

- Runtime invariant: enrichment on the representative 94-metabolite dataset completes in approximately 3 seconds — substantially faster than MetaboAnalystR (~28 s) and FELLA (~14 min) on the same input.
- Schema check: `results$pathway_enrichment_all` should report the number of pathways tested (e.g. 32 human KEGG pathways for a 10-metabolite example) and `results$pathway_enrichment_results` should contain only rows with P_value <= p_value_cutoff (default cutoff 1 in README example).
- MetSEA NES sign invariant: positive NES with adjusted p-value < 0.05 indicates significant positive pathway enrichment; negative NES with adjusted p-value < 0.05 indicates significant negative enrichment.
- Background correction check: when `backgroundMetabolites` is supplied, the Fisher's test universe should shrink to only metabolites measured in the experiment (testable set), reducing bias from database compounds never observed.
- Plot inventory check: the enrichmet() call should return eight plot objects (pathway_plot, impact_plot, gsea_plot, rbc_plot, network_plot, heatmap_plot, membership_plot, interaction_plot), plus a volcano plot if run_de() was used.

## Limitations

- Pathway and compound annotations are obtained at runtime from public APIs (KEGG REST, Reactome mappings, LION/BioPortal) and cached locally with BiocFileCache — the package does not redistribute third-party database dumps, so an internet connection is needed on first run and offline environments must supply equivalent Pathway/Metabolites tables.
- Lipid ontology content (LION) is not shipped with the package and must be fetched on demand, so lipidomics analyses depend on runtime API availability.
- No changelog is documented in the article's discussion section.
- The benchmark claim of ~3 seconds applies to a representative 94-metabolite dataset; runtime may vary with different input sizes or slower network fetches during annotation retrieval.

## Evidence

- [abstract] EnrichMet modules support over-representation analysis of pathways, metabolite set enrichment analysis (MetSEA), and network-based pathway analysis: "EnrichMet modules support over-representation analysis of pathways, metabolite set enrichment analysis (MetSEA), and network-based pathway analysis"
- [intro] Fisher’s Exact Test applied to contingency table with Benjamini-Hochberg adjustment: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [intro] STITCH interactions filtered at combined score >= 100: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] Significant metabolites filtered at absolute fold change >= 1 and adjusted p-value < 0.05: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [intro] Runtime benchmark: EnrichMet ~3 s, MetaboAnalystR ~28 s, FELLA ~14 min on 94-metabolite dataset: "EnrichMet completed the pathway analysis for representative dataset of 94 metabolites in approximately 3 seconds, compared with approximately 28 seconds for MetaboAnalystR and approximately 14"
- [readme] Annotations obtained at runtime from public APIs and cached with BiocFileCache: [evidence span withheld — anchor status: external_doc]
- [readme] Background-corrected enrichment restricts Fisher's test to measured metabolites: "Fisher’s exact test can be restricted to metabolites that were measured (testable) in the experiment, reducing bias from database compounds that were never observed."
- [readme] enrichmet() produces three tables and eight plots: "The `enrichmet()` function produces three tables (S3 `data.frame` objects): pathway enrichment results, MetSEA results, and metabolite centrality. In addition, it generates eight plots (S3/S4 plot"
