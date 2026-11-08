---
name: over-representation-analysis-fishers-exact-test
description: Use when you have a query set of metabolite/lipid identifiers and a pathway-to-feature
  mapping table, and need to identify which pathways are over-represented at adjusted
  p-value < 0.05.
when_to_use_negative:
- When you have ranked feature lists with log2 fold change and p-values for GSEA-style
  testing — use EnrichMet's MetSEA/fgsea module instead of Fisher's Exact Test ORA
- When the question is network/topology impact (betweenness centrality, pathway impact
  scores) rather than set over-representation
- When no pathway-to-feature mapping table (KEGG or user-supplied Pathway/Metabolites
  table) is available for the feature type in question
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: EnrichMet
  role: R package executing Fisher's exact test ORA in a single enrichmet() call,
    optionally background-corrected, producing enrichment tables and dot plots
  repo: https://github.com/biodatalab/enrichmet.git
- name: LION
  role: Source of lipid-ontology pathway-to-lipid associations enabling the same ORA
    framework to run on lipidomics query sets
  repo: http://www.lipidontology.com/
- name: R
  role: Runtime environment for EnrichMet; annotations fetched via KEGG REST and BioPortal/LION
    APIs and cached with BiocFileCache
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:240d32374ed2fa1047707357a06233fbfb7b6f41958d441fe29af776360f07c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - LION
  - R
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# over-representation-analysis-fishers-exact-test

## Summary

Performs pathway over-representation analysis (ORA) by applying Fisher's Exact Test to a 2x2 contingency table of query features (metabolites or lipids) mapped to pathways, with Benjamini-Hochberg adjustment; in EnrichMet this generic test is made applicable to lipidomics by swapping KEGG pathway maps for LION lipid-ontology associations.

## When to use

Use when you have a query set of metabolite/lipid identifiers and a pathway-to-feature mapping table, and need to identify which pathways are over-represented at adjusted p-value < 0.05.

## When NOT to use

- When you have ranked feature lists with log2 fold change and p-values for GSEA-style testing — use EnrichMet's MetSEA/fgsea module instead of Fisher's Exact Test ORA
- When the question is network/topology impact (betweenness centrality, pathway impact scores) rather than set over-representation
- When no pathway-to-feature mapping table (KEGG or user-supplied Pathway/Metabolites table) is available for the feature type in question

## Inputs

- Query set of metabolite/lipid identifiers (KEGG IDs, names, or mixed; e.g., lipid species like PA(18:0/16:0), TG(16:0/16:0/16:0), GlcCer(d18:0/24:0))
- Pathway-to-feature mapping table in Pathway/Metabolites format (KEGG hsa maps, or LION lipid-ontology associations from BioPortal/LION)
- Optional measured-metabolite background (e.g., extract_measured_kegg_ids(example_summary))
- Optional differential analysis results from run_de() with fold change and p-value thresholds

## Outputs

- Pathway enrichment results table with Pathway, P_value, Adjusted_P_value, and Enrichment_Ratio columns
- Dot plot of -log10(adjusted p-values) highlighting top significantly enriched pathways (pathway_plot)
- List of enriched pathways passing adjusted p-value < 0.05

## How to apply

Build a 2x2 contingency table counting query features in/out of each pathway against the background universe, then apply Fisher's Exact Test per pathway and adjust p-values with Benjamini-Hochberg, ranking pathways by -log10(adjusted p-value) and calling significance at adjusted p < 0.05. In EnrichMet, supply the mapping as a Pathway/Metabolites two-column table (e.g., KEGG hsa maps from fetch_kegg_pathway_metabolites) and optionally restrict the test to a measured background via backgroundMetabolites to reduce bias from database compounds never observed. Query sets can be KEGG IDs, names, or mixed (input_type = 'kegg'/'name'/'mixed'), or derived from run_de() output filtered at absolute fold change >= 1 and adjusted p < 0.05. For lipidomics extension, replace KEGG annotations with LION lipid-ontology associations fetched at runtime through the BioPortal/LION interface via fetch_lion_lipid_ontology(), and run the same Fisher 2x2 test on lipids mapped to LION terms, visualizing results as dot plots of -log10(adjusted p-values).

## Related tools

- **EnrichMet** (R package executing Fisher's exact test ORA in a single enrichmet() call, optionally background-corrected, producing enrichment tables and dot plots) — https://github.com/biodatalab/enrichmet.git
- **LION** (Source of lipid-ontology pathway-to-lipid associations enabling the same ORA framework to run on lipidomics query sets) — http://www.lipidontology.com/
- **R** (Runtime environment for EnrichMet; annotations fetched via KEGG REST and BioPortal/LION APIs and cached with BiocFileCache)

## Examples

```
PathwayVsMetabolites <- fetch_lion_lipid_ontology(); results <- enrichmet(inputMetabolites = lipid_list, PathwayVsMetabolites = PathwayVsMetabolites, input_type = "name")
```

## Evaluation signals

- Adjusted p-values computed via Benjamini-Hochberg, with enriched pathways called at adjusted p < 0.05
- Enrichment results and dot plot reproduce the lipidomics case study output (Figure S2) showing applicability beyond KEGG metabolomics
- Results table contains expected columns (Pathway, P_value, Adjusted_P_value, Enrichment_Ratio) and the number of pathways tested matches the size of the mapping table
- When a measured background is supplied, the contingency-table universe shrinks to testable metabolites and enrichment ratios shift accordingly
- Lipidomics run succeeds using LION mappings retrieved at runtime and cached locally (e.g., via BiocFileCache), without shipped database dumps

## Limitations

- The package does not redistribute third-party database dumps (KEGG, Reactome, or LION); annotations are fetched at runtime from public APIs, so network access or cache availability is required
- No changelog documented for the package
- ORA relies on a complete pathway-to-feature mapping; LION lipid ontology content is not shipped and must be fetched via fetch_lion_lipid_ontology() or supplied by the user

## Evidence

- [intro] Fisher's Exact Test on 2x2 contingency with BH adjustment: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [intro] Significant metabolite filter for ORA input: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [intro] Lipidomics extension source data: "Lipid–ontology associations of this analysis were sourced from the LION database [30]"
- [readme] ORA implementation detail in README: "**enrichmet** performs pathway over-representation analysis using Fisher’s exact test (optionally restricted to a measured background)"
- [readme] Generic pathway-feature interface enabling LION swap: "The same over-representation interface accepts any pathway–feature table in `Pathway` / `Metabolites` format."
- [readme] LION runtime fetch mechanism: "LION lipid ontology content is obtained at runtime through the BioPortal / LION programming interface when users call the corresponding package helper, such as `fetch_lion_lipid_ontology()`"
- [abstract] Abstract-level claim of the test: "employs Fisher’s Exact Test to identify significantly enriched pathways"
