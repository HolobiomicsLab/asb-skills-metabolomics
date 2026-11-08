---
name: hypergeometric-pathway-enrichment-testing
description: Use when a table of significantly altered metabolites with KEGG compound
  IDs must be associated with species-specific KEGG pathways and each pathway scored
  for over-representation with a hypergeometric p-value.
when_to_use_negative:
- Metabolites lack KEGG compound IDs — met_pathways retains only metabolites with
  linked KEGG reactions, so unmapped compounds silently drop out of the test
- The goal is metabolite-class (RefMet class) enrichment rather than pathway enrichment
  — use metclassenrichment instead of met_pathways/path_enrichmentscore
- The species of interest has no KEGG pathways available — MetENP supports only species
  with KEGG pathway entries
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetENP
  role: R package providing the met_pathways and path_enrichmentscore functions for
    pathway association and hypergeometric enrichment
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client used by met_pathways to link significant metabolites to
    KEGG reactions and species-specific KEGG pathways
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: Reference database supplying reactions, pathways, and organism-specific pathway
    maps used for the enrichment test
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:c8c621ac6dee9fee71453c036b09c84afe75ef6b487bdf28c4e978d9d2f18bd1
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# hypergeometric-pathway-enrichment-testing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

MetENP's pathway-association and enrichment step: significant metabolites (with KEGG compound IDs) are linked via KEGGREST to KEGG reactions and then to species-specific KEGG pathways, and each pathway is scored with a hypergeometric test (phyper) over-representation p-value.

## When to use

Use when a table of significantly altered metabolites with KEGG compound IDs must be associated with species-specific KEGG pathways and each pathway scored for over-representation with a hypergeometric p-value.

## When NOT to use

- Metabolites lack KEGG compound IDs — met_pathways retains only metabolites with linked KEGG reactions, so unmapped compounds silently drop out of the test
- The goal is metabolite-class (RefMet class) enrichment rather than pathway enrichment — use metclassenrichment instead of met_pathways/path_enrichmentscore
- The species of interest has no KEGG pathways available — MetENP supports only species with KEGG pathway entries

## Inputs

- Data frame of significant metabolites with KEGG compound IDs (e.g. from ST000915 on Metabolomics Workbench)
- Metabolite-class enrichment data frame (output of metclassenrichment)
- Species code (sps) for the KEGG organism of choice
- padj significance cutoff for metabolites within each pathway

## Outputs

- Per-pathway enrichment table (pathway ID/name, counts of significant vs total metabolites, enrichment score, hypergeometric p-value), e.g. st000915_pathway_enrichment_scores.csv

## How to apply

Load the significant-metabolite table (e.g. ST000915, with KEGG compound IDs) together with the metabolite-class enrichment data frame in R with MetENP and its Bioconductor dependencies (KEGGREST, KEGGgraph) attached. Call met_pathways(df_metenrichment, sps, debug), which queries KEGG via KEGGREST to associate significant metabolites with reactions and pathways of the chosen species (e.g. human, bacteria, dog), retaining only metabolites that have linked KEGG reactions. Then run path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps, padj, kegg_comp_path, sig_metabolites_kegg_id, debug), which applies the user-specified padj significance cutoff to metabolites within each pathway and computes each pathway's enrichment score and hypergeometric p-value via phyper. Write the per-pathway results (pathway ID/name, counts of significant vs total metabolites, enrichment score, p-value) to a CSV such as st000915_pathway_enrichment_scores.csv. The rationale is that reaction-level linkage anchors metabolites in the species' KEGG pathway graph before statistical testing, making enrichment species-specific as MetENP was designed for.

## Related tools

- **MetENP** (R package providing the met_pathways and path_enrichmentscore functions for pathway association and hypergeometric enrichment) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client used by met_pathways to link significant metabolites to KEGG reactions and species-specific KEGG pathways) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (Reference database supplying reactions, pathways, and organism-specific pathway maps used for the enrichment test)

## Examples

```
met_path <- met_pathways(df_metenrichment, sps="hsa", debug=TRUE); path_scores <- path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps="hsa", padj=0.05, kegg_comp_path, sig_metabolites_kegg_id, debug=TRUE)
```

## Evaluation signals

- Output CSV (e.g. st000915_pathway_enrichment_scores.csv) exists and contains one row per KEGG pathway with populated enrichment-score and p-value fields
- Metabolite counts per pathway reflect only metabolites that have linked KEGG reactions (unmapped compounds excluded from both significant and total counts)
- p-values are consistent with a phyper hypergeometric computation given the significant-vs-total metabolite counts and the padj cutoff applied
- All returned pathway IDs belong to the chosen species (sps) or the reference pathway set, confirming species-specific mapping via KEGGREST
- Pathways containing no metabolites passing the padj cutoff show no significant enrichment signal

## Limitations

- Only metabolites with linked KEGG reactions are retained; metabolites without reaction links are dropped before enrichment testing
- Support is limited to metabolite species with available KEGG pathways
- Results depend on the chosen padj significance cutoff, which the user must specify; no quantitative benchmark metrics were reported for the case studies

## Evidence

- [other] The met_pathways function links significant metabolites to KEGG reactions and then to KEGG pathways (species-specific or reference), and path_enrichmentscore computes pathway enrichment via a hypergeometric test using phyper: [evidence span withheld — anchor status: not_found]
- [readme] MetENP depends on following Bioconductor packages to function properly: KEGGREST, KEGGgraph, and pathview: "MetENP package depends on following Bioconductor packages to function properly: KEGGREST, KEGGgraph, and pathview."
