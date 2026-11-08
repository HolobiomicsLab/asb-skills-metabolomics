---
name: species-specific-reaction-annotation
description: Use when a metabolomics differential table already has KEGG compound
  IDs for significant metabolites and you need species-specific reaction-to-pathway
  associations before computing pathway enrichment p-values.
when_to_use_negative:
- Input metabolites lack KEGG compound IDs (use map_keggid/convert_refmet mapping
  steps first)
- The goal is metabolite-class (RefMet class) enrichment rather than pathway-level
  association
- A reference (non-species-specific) pathway mapping is desired instead of the species-of-choice
  KEGG pathways
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0092
tools:
- name: MetENP
  role: provides met_pathways and path_enrichmentscore functions for reaction/pathway
    association and hypergeometric enrichment scoring
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client used by met_pathways to query KEGG for metabolite-reaction-pathway
    links of the chosen species
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: reference and species-specific pathway database providing reaction and pathway
    definitions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:5ea3b73559a6ee063f3237ce5431a3f8c5ba43fd02fc297f365f056162ceb123
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# species-specific-reaction-annotation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Links significantly altered metabolites (carrying KEGG compound IDs) to KEGG reactions and then to KEGG pathways restricted to a chosen species using MetENP's met_pathways and KEGGREST, producing the pathway-association table that feeds hypergeometric pathway enrichment scoring.

## When to use

Use when a metabolomics differential table already has KEGG compound IDs for significant metabolites and you need species-specific reaction-to-pathway associations before computing pathway enrichment p-values.

## When NOT to use

- Input metabolites lack KEGG compound IDs (use map_keggid/convert_refmet mapping steps first)
- The goal is metabolite-class (RefMet class) enrichment rather than pathway-level association
- A reference (non-species-specific) pathway mapping is desired instead of the species-of-choice KEGG pathways

## Inputs

- Significant-metabolite table with KEGG compound IDs (e.g., ST000915 significant-metabolite CSV)
- Metabolite-class enrichment data frame (df_metenrichment from metclassenrichment)
- Species code (sps, e.g., human, bacteria, dog korg entries)
- Adjusted p-value cutoff (padj) for metabolites within each pathway

## Outputs

- Per-pathway enrichment table (pathway ID/name, counts of significant vs. total metabolites, enrichment score, hypergeometric p-value), e.g., st000915_pathway_enrichment_scores.csv
- met_path object of reaction-linked metabolite–pathway associations

## How to apply

Load the significant-metabolite table (e.g., from ST000915 on Metabolomics Workbench) with the MetENP package attached along with its Bioconductor dependencies KEGGREST and KEGGgraph. Call met_pathways(df_metenrichment, sps, debug), which queries KEGG via KEGGREST to associate each significant metabolite with reactions and pathways of the chosen species (sps); metabolites without linked KEGG reactions are dropped, so only reaction-linked compounds proceed. Feed the resulting met_path object into path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps, padj, kegg_comp_path, sig_metabolites_kegg_id, debug), which applies the padj significance cutoff to metabolites within each pathway and computes a hypergeometric enrichment p-value via phyper. Write the per-pathway results (pathway ID/name, counts of significant vs. total metabolites, enrichment score, p-value) to a CSV such as st000915_pathway_enrichment_scores.csv. Choose sps according to the study organism — MetENP supports all metabolite species with available KEGG pathways, validated in the paper for human, bacteria, and dog case studies.

## Related tools

- **MetENP** (provides met_pathways and path_enrichmentscore functions for reaction/pathway association and hypergeometric enrichment scoring) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client used by met_pathways to query KEGG for metabolite-reaction-pathway links of the chosen species) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (reference and species-specific pathway database providing reaction and pathway definitions)

## Examples

```
met_path <- met_pathways(df_metenrichment, sps="hsa", debug=FALSE); scores <- path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, padj=0.05, kegg_comp_path, sig_metabolites_kegg_id)
```

## Evaluation signals

- Output CSV (e.g., st000915_pathway_enrichment_scores.csv) exists and contains one row per KEGG pathway with populated enrichment-score and p-value fields
- Metabolite counts per pathway reflect only metabolites with linked KEGG reactions (non-reaction-linked compounds are excluded)
- Every p-value matches a hypergeometric (phyper) computation given the pathway's significant-vs-total metabolite counts
- All reported pathways belong to the specified species (sps) or the chosen reference pathway set
- Metabolites failing the padj significance cutoff are excluded from within-pathway counts

## Limitations

- Metabolites without linked KEGG reactions are silently dropped from pathway association, reducing coverage
- Requires network access to the KEGG REST API via KEGGREST; results depend on KEGG pathway availability for the chosen species
- No quantitative benchmark metrics were reported for the human, bacteria, and dog case studies, so output quality must be judged by the validation signals above

## Evidence

- [methods] met_pathways links significant metabolites to KEGG reactions and then to KEGG pathways (species-specific or reference): [evidence span withheld — anchor status: not_found]
- [methods] path_enrichmentscore computes pathway enrichment via a hypergeometric test using phyper: [evidence span withheld — anchor status: not_found]
