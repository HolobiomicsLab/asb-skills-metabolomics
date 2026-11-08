---
name: metabolomics-enrichment-analysis
description: Use when you have a metabolite table (names/concentrations plus metadata)
  from a comparative metabolomics experiment and need enrichment of metabolite classes
  and KEGG pathways for a species of choice.
when_to_use_negative:
- Your metabolites cannot be mapped to KEGG compound IDs or the species has no available
  KEGG pathways — MetENP supports only species with KEGG pathways
- You need gene-level or transcriptomic (e.g. GO/GSEA) enrichment rather than metabolite-class
  and KEGG pathway enrichment of altered metabolites
- You only have an already-computed list of enriched metabolites with no metadata/concentration
  information needed to detect significant metabolites
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetENP
  role: R package implementing significant-metabolite detection, metabolite-class
    enrichment, KEGG pathway association, and pathway enrichment scoring
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client used by met_pathways to associate significant metabolites
    with KEGG reactions and pathways of the chosen species
  repo: https://github.com/Bioconductor/KEGGREST
- name: MetENPweb
  role: Web application alternative on Metabolomics Workbench for running metabolite
    and pathway enrichment analysis without R
- name: RefMet
  role: Provides standardized metabolite nomenclature/classes used as the basis for
    metabolite-class enrichment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:4389b76a4bef6caf102e30abc762ee52c2fb35440f87144787ac03dbb4ee9146
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - MetENPweb
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# metabolomics-enrichment-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

MetENP-based metabolite-class and species-specific KEGG pathway enrichment analysis: detect significantly altered metabolites, compute class enrichment scores, map to KEGG pathways, and compute pathway enrichment via hypergeometric testing.

## When to use

Use when you have a metabolite table (names/concentrations plus metadata) from a comparative metabolomics experiment and need enrichment of metabolite classes and KEGG pathways for a species of choice.

## When NOT to use

- Your metabolites cannot be mapped to KEGG compound IDs or the species has no available KEGG pathways — MetENP supports only species with KEGG pathways
- You need gene-level or transcriptomic (e.g. GO/GSEA) enrichment rather than metabolite-class and KEGG pathway enrichment of altered metabolites
- You only have an already-computed list of enriched metabolites with no metadata/concentration information needed to detect significant metabolites

## Inputs

- Metabolite information table (metabolite names or names plus concentrations with metadata) from a Metabolomics Workbench study (e.g. ST000915) or custom dataset
- Species code (sps) for KEGG pathway mapping (e.g. human, bacteria, dog)
- Adjusted p-value cutoff (padj) for defining significantly altered metabolites
- RefMet-standardized metabolite classes and KEGG compound IDs per metabolite

## Outputs

- Per-pathway enrichment table (pathway ID/name, counts of significant vs total metabolites, enrichment score, hypergeometric p-value), e.g. st000915_pathway_enrichment_scores.csv
- Metabolite-class enrichment scores
- Plots of enriched pathways showing metabolite increase/decrease

## How to apply

Run the MetENP R workflow: first map metabolites to standardized RefMet classes (convert_refmet) and detect significantly altered metabolites (significant_met / anova_ana), then map KEGG compound IDs (map_keggid, compoundinfo). Call metclassenrichment to score metabolite-class enrichment, then met_pathways(df_metenrichment, sps, debug), which uses KEGGREST to link significant metabolites to KEGG reactions and species-specific (or reference) KEGG pathways, retaining only metabolites with linked reactions. Finally run path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps, padj, kegg_comp_path, sig_metabolites_kegg_id, debug), which applies the user-specified padj significance cutoff and computes per-pathway enrichment scores and hypergeometric p-values via phyper. Install MetENP via devtools (with Bioconductor dependencies KEGGREST, KEGGgraph, pathview) or via the provided conda environment.yml.

## Related tools

- **MetENP** (R package implementing significant-metabolite detection, metabolite-class enrichment, KEGG pathway association, and pathway enrichment scoring) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client used by met_pathways to associate significant metabolites with KEGG reactions and pathways of the chosen species) — https://github.com/Bioconductor/KEGGREST
- **MetENPweb** (Web application alternative on Metabolomics Workbench for running metabolite and pathway enrichment analysis without R)
- **RefMet** (Provides standardized metabolite nomenclature/classes used as the basis for metabolite-class enrichment)

## Examples

```
met_path <- met_pathways(df_metenrichment, sps="hsa", debug=TRUE); path_scores <- path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps="hsa", padj=0.05, kegg_comp_path, sig_metabolites_kegg_id, debug=TRUE)
```

## Evaluation signals

- The pathway enrichment output CSV (e.g. st000915_pathway_enrichment_scores.csv) exists and contains one row per KEGG pathway with populated enrichment-score and p-value fields
- Metabolite counts per pathway reflect only metabolites that have linked KEGG reactions (met_pathways drops metabolites without linked reactions)
- Pathway p-values are hypergeometric (phyper) values within [0,1], with enrichment scores consistent with the ratio of significant to total metabolites in each pathway
- Every significant metabolite passed the user-specified padj cutoff and carries a valid RefMet class and KEGG compound ID before enrichment scoring
- Class and pathway enrichment results match the MetENP vignette/MetENPWeb output when run on the same study (e.g. ST000915) with the same parameters

## Limitations

- Species coverage is limited to metabolite species with available KEGG pathways; mapping uses species-specific or reference KEGG pathways
- Only metabolites that have linked KEGG reactions are retained for pathway association, so metabolites without reaction links are excluded from pathway enrichment
- No software version number, changelog, or explicit statistical validation metrics for the case studies (Human, bacteria, dog) are reported

## Evidence

- [methods] met_pathways links significant metabolites to KEGG reactions and then to KEGG pathways (species-specific or reference): [evidence span withheld — anchor status: not_found]
