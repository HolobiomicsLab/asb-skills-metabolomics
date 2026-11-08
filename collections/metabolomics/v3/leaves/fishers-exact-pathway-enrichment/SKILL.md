---
name: fishers-exact-pathway-enrichment
description: Use when you have a set of significant metabolites (e.g. absolute fold
  change >= 1 and adjusted p < 0.
when_to_use_negative:
- When the goal is ranking based on effect direction/magnitude rather than over-representation
  — use EnrichMet's MetSEA (fgsea on ranked log2 fold change and p-value lists with
  GMT-formatted annotations) instead.
- When the query is already a pathway topology/impact problem (betweenness centrality,
  network connectivity) — EnrichMet's topology-based network analysis module is the
  appropriate choice.
- When input is a raw metabolomics matrix without differential statistics — first
  run run_de() with fc_threshold = 1 and pval_threshold = 0.05 to obtain the significant
  metabolite set.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: EnrichMet
  role: R package providing the Fisher's Exact Test over-representation module, runtime
    KEGG annotation fetching, Benjamini-Hochberg adjustment, and the pathway enrichment
    table/dot plot outputs
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Language environment in which the EnrichMet over-representation workflow is
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:4e58a0abc3d3555ca93706c15015b7fbc0928e132b4c9468e2beb1453ec7984b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fishers-exact-pathway-enrichment

## Summary

Over-representation analysis of metabolomics data via Fisher's Exact Test on 2x2 contingency tables per KEGG pathway, with Benjamini-Hochberg p-value adjustment, as implemented in the EnrichMet R package to identify significantly enriched metabolic pathways from a set of significant metabolites.

## When to use

Use when you have a set of significant metabolites (e.g. absolute fold change >= 1 and adjusted p < 0.05) from differential metabolomics analysis and need to rank KEGG pathways by over-rerepresentation to identify which are biologically enriched, as in the EnrichMet control-vs-mutant lung cancer case study.

## When NOT to use

- When the goal is ranking based on effect direction/magnitude rather than over-representation — use EnrichMet's MetSEA (fgsea on ranked log2 fold change and p-value lists with GMT-formatted annotations) instead.
- When the query is already a pathway topology/impact problem (betweenness centrality, network connectivity) — EnrichMet's topology-based network analysis module is the appropriate choice.
- When input is a raw metabolomics matrix without differential statistics — first run run_de() with fc_threshold = 1 and pval_threshold = 0.05 to obtain the significant metabolite set.

## Inputs

- Metabolomics summary statistics per metabolite (met_id, log2 fold change, p-values) from differential analysis (e.g. run_de() output or summary_stat.csv)
- Query set of significant metabolites as KEGG IDs and/or compound names (input_type = "kegg"/"name"/"mixed")
- Pathway-to-metabolite mapping table (Pathway / Metabolites columns) from KEGG, e.g. fetch_kegg_pathway_metabolites(organism = "hsa")
- Optional measured-metabolite background (extract_measured_kegg_ids(example_summary))

## Outputs

- Enrichment table with columns Pathway, P_value, Adjusted_P_value, Enrichment_Ratio for each tested KEGG pathway
- Dot plot of -log10(adjusted p-values) highlighting the top significantly enriched pathways (pathway_plot)
- Ranked pathway list ordered by adjusted p-value (e.g. ora_pathway_enrichment_table.csv)

## How to apply

Load the control-vs-mutant lung cancer metabolomics summary statistics (log2 fold change and p-values per metabolite) into R and select significant metabolites using EnrichMet thresholds of absolute fold change >= 1 and adjusted p-value < 0.05. Fetch the human KEGG pathway-to-metabolite map at runtime via fetch_kegg_pathway_metabolites(organism = "hsa") and build a 2x2 contingency table for each pathway counting the significant metabolites vs the pathway's total metabolites. Apply Fisher's Exact Test per pathway via EnrichMet's over-representation module, optionally restricting the testable background to metabolites actually measured (extract_measured_kegg_ids) to reduce bias from database compounds never observed, and adjust p-values with Benjamini-Hochberg. Rank pathways by adjusted p-value, visualise the top hits with dot plots of -log10(adjusted p-values) (as in Figure 2A, where pyrimidine metabolism ranks first for the lung cancer case study), and export the enrichment table with columns Pathway, P_value, Adjusted_P_value, and Enrichment_Ratio.

## Related tools

- **EnrichMet** (R package providing the Fisher's Exact Test over-representation module, runtime KEGG annotation fetching, Benjamini-Hochberg adjustment, and the pathway enrichment table/dot plot outputs) — https://github.com/biodatalab/enrichmet.git
- **R** (Language environment in which the EnrichMet over-representation workflow is executed)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, backgroundMetabolites = extract_measured_kegg_ids(example_summary), input_type = "kegg")
```

## Evaluation signals

- Pathway ranking matches the published Figure 2A ordering for the lung cancer case study, with pyrimidine metabolism as the top pathway by adjusted p-value.
- Enrichment table schema contains Pathway, P_value, Adjusted_P_value, and Enrichment_Ratio columns and reports only pathways with adjusted p-value < 0.05.
- Every pathway's 2x2 contingency table is constructed from metabolites mapped to KEGG IDs (e.g. C00031, C00022, C00197) under the fetch_kegg_pathway_metabolites('hsa') mapping.
- If a measured background is supplied, tested pathways are restricted to metabolites actually observed (extract_measured_kegg_ids), and enrichment ratios reflect that restricted background.
- Runtime of the over-representation analysis is on the order of seconds (approximately 3 seconds on the 94-metabolite benchmark), far faster than MetaboAnalystR (~28 s) or FELLA (~14 min).

## Limitations

- Bias arises if Fisher's Exact Test uses all KEGG database compounds rather than a measured background; restrict backgroundMetabolites to metabolites actually measured to reduce this.
- Pathway and compound annotations are obtained at runtime from KEGG REST and cached with BiocFileCache — no offline/dump redistribution, so network access or user-supplied tables are required.
- The approach is demonstrated on KEGG pathway data; extension to other knowledge bases (e.g. LION for lipidomics) requires supplying an equivalent Pathway/Metabolites table via runtime fetchers like fetch_lion_lipid_ontology().

## Evidence

- [intro] Fisher's Exact Test applied to a contingency table with Benjamini-Hochberg adjustment for pathway over-representation analysis: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [intro] Significant metabolite threshold: absolute fold change >= 1 and adjusted p-value < 0.05: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher’s Exact Test for pathway enrichment analysis"
- [abstract] EnrichMet employs Fisher's Exact Test to identify significantly enriched pathways: "employs Fisher’s Exact Test to identify significantly enriched pathways"
- [readme] README documents Fisher's exact test over-representation with optional measured background and pathway_plot output: "**enrichmet** performs pathway over-representation analysis using
Fisher’s exact test (optionally restricted to a measured background)"
- [readme] README example shows enrichment results table columns Pathway, P_value, Adjusted_P_value, Enrichment_Ratio: "results$pathway_enrichment_results[
    , c("Pathway", "P_value", "Adjusted_P_value", "Enrichment_Ratio")
  ]"
