---
name: refmet-nomenclature-mapping
description: Use when metabolite identifiers in a study's metabolite table are non-standard
  (free-text names) and must be harmonized to RefMet and KEGG compound IDs before
  counting significant metabolites or mapping to KEGG pathways.
when_to_use_negative:
- Input metabolite identifiers are already RefMet-standard names and KEGG IDs are
  already attached — re-running convert_refmet() adds no information
- The target database is not KEGG (e.g., analysis targets Reactome or MetaCyc pathways),
  since map_keggid()/compoundinfo() produces KEGG compound IDs
- Working with MetENPweb only and no command-line/custom dataset processing is needed,
  since the web interface handles standardization internally
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MetENP
  role: R package supplying convert_refmet(), map_keggid(), and compoundinfo() for
    RefMet standardization and KEGG compound ID mapping
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: RefMet
  role: Reference nomenclature database providing standard metabolite names that enable
    comparisons across experiments
- name: KEGGREST
  role: Bioconductor client interface to the KEGG REST API used to resolve compound
    IDs for the species of choice
  repo: https://github.com/Bioconductor/KEGGREST
- name: Metabolomics Workbench
  role: Source of the metabolite and metadata tables that are standardization targets
    (getmwstudies())
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:28c705831330ec6927096bf6d84157da9e5055ceae89a2330a81ddb94d96e359
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - Metabolomics Workbench
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# refmet-nomenclature-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardizes metabolite names from a metabolomics dataset (e.g., Metabolomics Workbench study tables) to RefMet nomenclature using MetENP's convert_refmet(), then maps standardized names to species-specific KEGG compound IDs via map_keggid()/compoundinfo() so downstream enrichment and pathway analysis can proceed.

## When to use

Use when metabolite identifiers in a study's metabolite table are non-standard (free-text names) and must be harmonized to RefMet and KEGG compound IDs before counting significant metabolites or mapping to KEGG pathways.

## When NOT to use

- Input metabolite identifiers are already RefMet-standard names and KEGG IDs are already attached — re-running convert_refmet() adds no information
- The target database is not KEGG (e.g., analysis targets Reactome or MetaCyc pathways), since map_keggid()/compoundinfo() produces KEGG compound IDs
- Working with MetENPweb only and no command-line/custom dataset processing is needed, since the web interface handles standardization internally

## Inputs

- Metabolite table (metabolite names or names plus concentrations) from a Metabolomics Workbench study, e.g. retrieved with getmwstudies(studyid, typedata)
- Metadata table with treatment/factor information (e.g., prednisolone vs tetracosactide)
- Species/organism identifier for KEGG mapping (e.g., 'dga' for dog)

## Outputs

- Metabolite table with RefMet-standardized names (output of convert_refmet())
- Metabolite-to-KEGG compound ID mapping (output of map_keggid()/compoundinfo())
- Named list/annotations used as input to significant_met() and met_pathways()

## How to apply

First retrieve the metabolite and metadata tables for the study of interest (e.g., via MetENP's getmwstudies(studyid, typedata) for a Metabolomics Workbench study such as ST001140, pulling all analytical method types). Run convert_refmet() on the metabolite name column to obtain RefMet standard names, which enables comparisons across experiments. Then use map_keggid()/compoundinfo() with the organism code of the species of choice (e.g., 'dga' for dog, as in the canine plasma lipidomics case study) to attach KEGG compound IDs. The resulting name-to-KEGG mapping feeds significant_met() and met_pathways(); if the mapping rate is poor, significant metabolites will be lost downstream, so verify coverage before proceeding.

## Related tools

- **MetENP** (R package supplying convert_refmet(), map_keggid(), and compoundinfo() for RefMet standardization and KEGG compound ID mapping) — https://github.com/metabolomicsworkbench/MetENP
- **RefMet** (Reference nomenclature database providing standard metabolite names that enable comparisons across experiments)
- **KEGGREST** (Bioconductor client interface to the KEGG REST API used to resolve compound IDs for the species of choice) — https://github.com/Bioconductor/KEGGREST
- **Metabolomics Workbench** (Source of the metabolite and metadata tables that are standardization targets (getmwstudies()))

## Examples

```
source('path/convert_refmet.R'); source('path/map_keggid.R'); refmet <- convert_refmet(metab_table); kegg_ids <- map_keggid(refmet, species='dga')
```

## Evaluation signals

- Every input metabolite name should resolve to a RefMet standard name or an explicit unmatched category; unmatched names should be reviewed rather than silently dropped
- Output must contain a non-empty KEGG compound ID column for downstream met_pathways(); a near-zero mapping rate signals a nomenclature or species-code problem
- Species code passed to mapping must match the study organism (e.g., 'dga' for the canine ST001140 case study), verifiable against the korg organism list in MetENP
- Downstream validation: the mapped metabolites must be sufficient for the pipeline to recover the expected significant-metabolite and pathway counts (e.g., 26 significant metabolites and 7 KEGG pathways in the ST001140 case study)
- RefMet standardization should make names comparable across experiments — check that the same compound measured in different studies maps to the same standard name

## Limitations

- Mapping depends on KEGG pathway availability: MetENP includes support only for metabolite species with available KEGG pathways
- The article/README report no version, changelog, or statistical validation metrics for the standardization step itself; correctness rests on the case-study reproductions (human, bacteria, dog)
- Unstandardizable or non-KEGG-annotated metabolites (e.g., lipids without KEGG compound entries) will be lost before pathway enrichment, potentially reducing pathway counts

## Evidence

- [other] Standardize metabolite names with convert_refmet() and map them to KEGG compound IDs via map_keggid()/compoundinfo() using the dog (dga) species.: [evidence span withheld — anchor status: not_found]
