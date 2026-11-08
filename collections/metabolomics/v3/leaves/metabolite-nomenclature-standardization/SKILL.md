---
name: metabolite-nomenclature-standardization
description: Use when an input metabolomics dataset carries raw or inconsistently
  formatted metabolite names (e.g. from a Metabolomics Workbench study via getmwstudies)
  that must be mapped to RefMet standard classes before enrichment scoring.
when_to_use_negative:
- When metabolites are already keyed by RefMet-standard names or existing class annotations
  — the mapping step would be redundant re-processing
- When the goal is KEGG-based pathway mapping rather than metabolite-class standardization;
  use map_keggid / met_pathways instead
- When metabolite identifiers are already structured IDs (e.g. KEGG compound IDs)
  rather than free-text names, since convert_refmet targets name-to-nomenclature mapping
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetENP
  role: Provides the convert_refmet.R function that maps metabolite names to RefMet
    standard nomenclature and its class hierarchy
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: RefMet
  role: Reference database supplying the standardized metabolite nomenclature and
    main_class/sub_class/super_class categories assigned to each metabolite
- name: R
  role: Execution environment in which the MetENP package and convert_refmet are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:923cc2de2d5cc966c7ad841f2032bfc5b66bd32b1213078413cd5d9cc54d7aeb
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolite-nomenclature-standardization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardizes free-text metabolite names by mapping them to RefMet's hierarchical classification (main_class, sub_class, super_class) using MetENP's convert_refmet function, so downstream metabolite-class enrichment analysis operates on a consistent nomenclature.

## When to use

Use when an input metabolomics dataset carries raw or inconsistently formatted metabolite names (e.g. from a Metabolomics Workbench study via getmwstudies) that must be mapped to RefMet standard classes before enrichment scoring.

## When NOT to use

- When metabolites are already keyed by RefMet-standard names or existing class annotations — the mapping step would be redundant re-processing
- When the goal is KEGG-based pathway mapping rather than metabolite-class standardization; use map_keggid / met_pathways instead
- When metabolite identifiers are already structured IDs (e.g. KEGG compound IDs) rather than free-text names, since convert_refmet targets name-to-nomenclature mapping

## Inputs

- Metabolite-name data frame retrieved from a Metabolomics Workbench study via getmwstudies(studyid, typedata), e.g. study ST000915
- Optional: user-supplied metabolite name/concentration table with metadata (the MetENP package accepts metabolite information as names or names plus concentrations)

## Outputs

- Annotated data frame / CSV (e.g. refmet_annotations.csv) with one row per input metabolite carrying RefMet main_class, sub_class, and super_class columns

## How to apply

Load the MetENP R package (or source('path/convert_refmet.R') when running from source) and explicitly attach KEGGREST since Imports dependencies are not auto-attached. Retrieve the metabolite-name data frame with getmwstudies(studyid, typedata) — the validated example used Metabolomics Workbench study ST000915. Pass the metabolite-name data frame to convert_refmet(metdata), which queries RefMet's standardized nomenclature and attaches the three hierarchical category columns — main_class, sub_class, and super_class — to each metabolite row. The rationale is that RefMet provides standard nomenclature enabling efficient comparisons between metabolite data across experiments, and this standardized class mapping is step (a) of the MetENP workflow before significant-metabolite detection and class enrichment scoring. Write the fully annotated table (e.g. refmet_annotations.csv) and verify that every input metabolite row carries non-missing main_class, sub_class, and super_class values before proceeding to metclassenrichment.

## Related tools

- **MetENP** (Provides the convert_refmet.R function that maps metabolite names to RefMet standard nomenclature and its class hierarchy) — https://github.com/metabolomicsworkbench/MetENP
- **RefMet** (Reference database supplying the standardized metabolite nomenclature and main_class/sub_class/super_class categories assigned to each metabolite)
- **R** (Execution environment in which the MetENP package and convert_refmet are run)

## Examples

```
source('path/convert_refmet.R'); refmet_ann <- convert_refmet(metdata); write.csv(refmet_ann, 'refmet_annotations.csv')
```

## Evaluation signals

- Every input metabolite row in the output (e.g. refmet_annotations.csv) has non-missing main_class, sub_class, and super_class fields — complete RefMet class annotation coverage
- The number of rows in the annotated output equals the number of metabolite rows in the input data frame (no metabolites dropped or duplicated)
- Assigned class values match RefMet vocabulary (valid main_class/sub_class/super_class terms), not the original free-text names
- Running on the reference input (Metabolomics Workbench study ST000915 via getmwstudies) reproduces the expected class annotations without errors from KEGGREST/httr calls
- Downstream metclassenrichment accepts the annotated table without requiring additional name cleanup

## Limitations

- MetENP is an Imports (not Suggests) consumer of KEGGREST, so the dependency must be attached explicitly; missing Bioconductor packages (KEGGREST, KEGGgraph, pathview) cause installation/runtime failures
- No software version number or changelog is stated for MetENP, so reproducibility of RefMet mapping results across package versions is not guaranteed
- The function depends on RefMet's external nomenclature service, so metabolite names that RefMet cannot recognize may fail to receive complete class annotations

## Evidence
