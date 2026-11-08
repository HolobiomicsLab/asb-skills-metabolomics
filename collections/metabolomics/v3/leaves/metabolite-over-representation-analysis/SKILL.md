---
name: metabolite-over-representation-analysis
description: Use when you have a list of significant metabolites (e.g. absolute fold
  change >= 1 and adjusted p-value < 0.05 from differential metabolomics) and need
  to rank KEGG pathways by enrichment significance.
when_to_use_negative:
- You want to rank metabolites by full distribution statistics rather than a significant-metabolite
  list — use MetSEA (fgsea on ranked log2 fold change / p-value lists) instead of
  Fisher's Exact Test ORA.
- You lack KEGG annotations for your metabolites and cannot supply an equivalent Pathway/Metabolites
  table — the Fisher's test depends on a pathway-feature mapping.
- You need lipid-ontology or other non-KEGG enrichment without first fetching the
  corresponding ontology mapping (e.g. fetch_lion_lipid_ontology()).
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: EnrichMet
  role: 'R package providing the over-representation module: builds 2x2 contingency
    tables per KEGG pathway, runs Fisher''s Exact Test with Benjamini-Hochberg adjustment,
    and generates enrichment/impact plots in a single function call'
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Runtime environment for EnrichMet; installed via BiocManager
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:62a22683375e42df62898c0e42327684133346898a54078d6a45b236ea1f07dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-over-representation-analysis

## Summary

Over-representation analysis of significant metabolites against KEGG pathways using Fisher's Exact Test on a 2x2 contingency table with Benjamini-Hochberg adjustment, implemented via the EnrichMet R package's enrichment module.

## When to use

Use when you have a list of significant metabolites (e.g. absolute fold change >= 1 and adjusted p-value < 0.05 from differential metabolomics) and need to rank KEGG pathways by enrichment significance.

## When NOT to use

- You want to rank metabolites by full distribution statistics rather than a significant-metabolite list — use MetSEA (fgsea on ranked log2 fold change / p-value lists) instead of Fisher's Exact Test ORA.
- You lack KEGG annotations for your metabolites and cannot supply an equivalent Pathway/Metabolites table — the Fisher's test depends on a pathway-feature mapping.
- You need lipid-ontology or other non-KEGG enrichment without first fetching the corresponding ontology mapping (e.g. fetch_lion_lipid_ontology()).

## Inputs

- Summary statistics table (CSV) with columns met_id, pval/padj, log2fc per metabolite
- Character vector or list of significant metabolites as KEGG IDs and/or names (input_type = "kegg", "name", or "mixed")
- Pathway-vs-metabolites mapping table (KEGG organism code, e.g. "hsa")
- KEGG compound ID/name lookup table (fetch_kegg_compound_lookup())
- Optional: measured-metabolite background (extract_measured_kegg_ids(example_summary))

## Outputs

- Pathway enrichment results table (data.frame) with Pathway, P_value, Adjusted_P_value, Enrichment_Ratio columns (e.g. ora_pathway_enrichment_table.csv)
- Ranked list of pathways by adjusted p-value (e.g. pyrimidine metabolism as top pathway in the lung cancer case study)
- Pathway enrichment dot plot of -log10(adjusted p-values) (e.g. figure2a_ora_dotplot.png)
- Pathway impact plot (significance vs topology-based impact score)

## How to apply

Load the control-vs-mutant lung cancer metabolomics summary statistics (log2 fold change and p-values per metabolite) into R and select significant metabolites using the EnrichMet thresholds of absolute fold change >= 1 and adjusted p-value < 0.05 (e.g. via run_de() with fc_threshold = 1, pval_threshold = 0.05). Fetch KEGG pathway-to-metabolite annotations at runtime with fetch_kegg_pathway_metabolites(organism = "hsa") and build a 2x2 contingency table for each pathway. Apply Fisher's Exact Test per pathway via the EnrichMet over-representation module, optionally restricting the test to the measured-metabolite background via extract_measured_kegg_ids() to reduce bias from database compounds never observed, and adjust p-values with Benjamini-Hochberg. Rank pathways by adjusted p-value, export the enrichment table (e.g. ora_pathway_enrichment_table.csv), and visualize with a dot plot of -log10(adjusted p-values) highlighting the top significantly enriched pathways. Optionally compute the topology-based pathway impact score alongside significance for the impact plot.

## Related tools

- **EnrichMet** (R package providing the over-representation module: builds 2x2 contingency tables per KEGG pathway, runs Fisher's Exact Test with Benjamini-Hochberg adjustment, and generates enrichment/impact plots in a single function call) — https://github.com/biodatalab/enrichmet.git
- **R** (Runtime environment for EnrichMet; installed via BiocManager)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = fetch_kegg_pathway_metabolites(organism = "hsa"), example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, backgroundMetabolites = extract_measured_kegg_ids(example_summary), input_type = "kegg")
```

## Evaluation signals

- The enrichment table contains columns Pathway, P_value, Adjusted_P_value, and Enrichment_Ratio for every tested pathway.
- Ranked ordering matches the reported result: pyrimidine metabolism is the top pathway by adjusted p-value in the control-vs-mutant lung cancer case study (Figure 2A/B).
- Input metabolite filtering reproduces the EnrichMet thresholds: absolute fold change >= 1 and adjusted p-value < 0.05.
- All reported pathways satisfy the significance cutoff (p <= p_value_cutoff, e.g. adjusted p < 0.05), and results were produced with Benjamini-Hochberg adjustment.
- Replication completes on the order of seconds (EnrichMet benchmark: ~3 s on a 94-metabolite dataset), not minutes, confirming the local Fisher's-test path rather than a slower external tool.

## Limitations

- Fisher's Exact Test over all KEGG compounds can be biased by database compounds never observed in the experiment; restrict to extract_measured_kegg_ids(example_summary) as backgroundMetabolites when a measured background is available.
- Results depend on runtime-fetched KEGG/Reactome annotations (KEGG REST, cached with BiocFileCache) — the package does not redistribute database dumps, so API availability and cache state affect reproducibility.
- ORA discards direction and magnitude information from the full ranked metabolite list; use the MetSEA (fgsea) module when NES direction matters.

## Evidence

- [intro] Pathway enrichment analysis (PEA) / over-representation analysis via Fisher's Exact Test on a 2x2 contingency table with Benjamini-Hochberg adjustment: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [intro] Significant metabolites filtered at absolute fold change >= 1 and adjusted p-value < 0.05: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [results] Case study finding: pyrimidine metabolism top-ranked pathway: [evidence span withheld — anchor status: not_found]
- [readme] Background-corrected enrichment option: "Fisher’s exact test can be restricted to metabolites that were measured (testable) in the experiment, reducing bias from database compounds that were never observed."
- [readme] Runtime annotation fetching from KEGG REST: [evidence span withheld — anchor status: not_found]
