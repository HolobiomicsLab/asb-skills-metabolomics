---
name: kegg-pathway-mapping
description: Use when a table of significant metabolites carrying KEGG compound IDs
  must be associated with KEGG pathways for a chosen species and ranked by enrichment
  p-value, as in the MetENP ST000915 workflow.
when_to_use_negative:
- Input metabolites lack KEGG compound IDs or cannot be reaction-linked — met_pathways
  retains only metabolites with linked KEGG reactions, so counts would be empty or
  misleading.
- The species of interest has no entry in MetENP's korg / no available KEGG pathways
  — the package supports only 'all metabolite species with available KEGG pathways'.
- The goal is metabolite-class (RefMet category) enrichment rather than pathway association
  — use metclassenrichment/plot_met_enrichment instead.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3391
tools:
- name: MetENP
  role: Provides met_pathways and path_enrichmentscore functions that perform the
    KEGG pathway mapping and hypergeometric pathway enrichment
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client used by met_pathways to associate significant metabolites
    with KEGG reactions and pathways of the chosen species
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: Reference/source database of reactions and pathways to which significant metabolites
    are mapped
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:c31cfaaff1e98ca687fdf90164f06c154ae3725cd5dbb029eae88067da795026
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- kegg-pathway-annotation-mapping
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# kegg-pathway-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Links significantly altered metabolites (with KEGG compound IDs) to species-specific KEGG reactions and pathways via MetENP's met_pathways (KEGGREST-based), then computes per-pathway enrichment scores and hypergeometric p-values with path_enrichmentscore.

## When to use

Use when a table of significant metabolites carrying KEGG compound IDs must be associated with KEGG pathways for a chosen species and ranked by enrichment p-value, as in the MetENP ST000915 workflow.

## When NOT to use

- Input metabolites lack KEGG compound IDs or cannot be reaction-linked — met_pathways retains only metabolites with linked KEGG reactions, so counts would be empty or misleading.
- The species of interest has no entry in MetENP's korg / no available KEGG pathways — the package supports only 'all metabolite species with available KEGG pathways'.
- The goal is metabolite-class (RefMet category) enrichment rather than pathway association — use metclassenrichment/plot_met_enrichment instead.

## Inputs

- Significant-metabolite table with KEGG compound IDs (e.g. ST000915 significant-metabolite CSV)
- Metabolite-class enrichment data frame (output of metclassenrichment)
- Species code (sps, e.g. human, bacteria, dog) resolvable via MetENP's korg
- Significance cutoff for adjusted p-values (padj)
- List of KEGG pathways (ls_path) and reference metabolite class annotations (refmet_class)

## Outputs

- Per-pathway enrichment table (pathway ID/name, counts of significant vs total metabolites, enrichment score, hypergeometric p-value), e.g. st000915_pathway_enrichment_scores.csv

## How to apply

Load the significant-metabolite table (e.g. ST000915) with KEGG compound IDs and the metabolite-class enrichment data frame into R with MetENP and its Bioconductor dependencies (KEGGREST, KEGGgraph) attached. Call met_pathways(df_metenrichment, sps, debug), which uses KEGGREST to associate each significant metabolite with KEGG reactions and then pathways of the chosen species (species-specific or reference), retaining only metabolites that have linked KEGG reactions. Then run path_enrichmentscore(met_path, sig_metabolite_kegg_id, ls_path, refmet_class, sps, padj, kegg_comp_path, sig_metabolites_kegg_id, debug), which computes each pathway's enrichment score and hypergeometric p-value via phyper, applying the user-specified padj cutoff to metabolites within each pathway. Write the per-pathway results (pathway ID/name, counts of significant vs total metabolites, enrichment score, p-value) to a CSV such as st000915_pathway_enrichment_scores.csv.

## Related tools

- **MetENP** (Provides met_pathways and path_enrichmentscore functions that perform the KEGG pathway mapping and hypergeometric pathway enrichment) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client used by met_pathways to associate significant metabolites with KEGG reactions and pathways of the chosen species) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (Reference/source database of reactions and pathways to which significant metabolites are mapped)

## Examples

```
met_path <- met_pathways(df_metenrichment, sps = "hsa", debug = TRUE)
```

## Evaluation signals

- Output CSV exists and contains one row per KEGG pathway with populated enrichment-score and p-value fields.
- Metabolite counts per pathway reflect only reaction-linked metabolites (metabolites without linked KEGG reactions are excluded by met_pathways).
- p-values are consistent with a hypergeometric (phyper) test given the significant-vs-total metabolite counts and the pathway background.
- The padj cutoff supplied to path_enrichmentscore is actually applied to metabolites within each pathway before counting.
- Returned pathway IDs/names correspond to the species specified via sps (species-specific or reference KEGG pathways).

## Limitations

- Only metabolites with linked KEGG reactions are retained, so metabolites lacking reaction annotations silently drop out of pathway counts.
- Support is limited to species with available KEGG pathways; the chosen sps must be resolvable (korg) and KEGG REST must be reachable at run time.
- Requires MetENP's Bioconductor dependencies (KEGGREST, KEGGgraph, pathview); installation may fail without them (install via BiocManager or the provided conda environment.yml).

## Evidence

- [methods] The met_pathways function links significant metabolites to KEGG reactions and then to KEGG pathways (species-specific or reference): [evidence span withheld — anchor status: not_found]
- [methods] path_enrichmentscore computes pathway enrichment via a hypergeometric test using phyper: [evidence span withheld — anchor status: not_found]
