---
name: hierarchical-metabolite-classification-mapping
description: Use when a metabolomics input table contains raw or non-standardized
  metabolite names (e.g. from a Metabolomics Workbench study) and downstream class-level
  enrichment analysis requires RefMet main_class/sub_class/super_class annotations.
when_to_use_negative:
- The input metabolite list already carries RefMet main_class/sub_class/super_class
  annotations, making re-mapping redundant
- The goal is pathway-level (KEGG) rather than metabolite-class-level analysis — use
  map_keggid/met_pathways instead
- Metabolite names are custom identifiers not mappable to RefMet standard nomenclature
edam_operation: http://edamontology.org/operation_3351
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetENP
  role: Provides the convert_refmet() function that performs the RefMet class mapping;
    loaded via devtools::install("MetENP") or sourced as path/convert_refmet.R
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: RefMet
  role: External reference resource supplying the standardized metabolite nomenclature
    and the main_class/sub_class/super_class hierarchy that convert_refmet maps names
    to
- name: KEGGREST
  role: Bioconductor dependency of MetENP that must be explicitly library()-attached
    before running convert_refmet because Imports packages are not attached automatically
  repo: https://github.com/Bioconductor/KEGGREST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:1a132483a8a62ec89a7b4fd34dbd62926689ea0cd14990a3083d183d7dd41f25
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_3351
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 24
---

# hierarchical-metabolite-classification-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Maps metabolite names in a metabolomics dataset to RefMet standardized nomenclature, extracting the hierarchical main_class, sub_class, and super_class categories used by MetENP's downstream metabolite-class enrichment steps.

## When to use

Use when a metabolomics input table contains raw or non-standardized metabolite names (e.g. from a Metabolomics Workbench study) and downstream class-level enrichment analysis requires RefMet main_class/sub_class/super_class annotations.

## When NOT to use

- The input metabolite list already carries RefMet main_class/sub_class/super_class annotations, making re-mapping redundant
- The goal is pathway-level (KEGG) rather than metabolite-class-level analysis — use map_keggid/met_pathways instead
- Metabolite names are custom identifiers not mappable to RefMet standard nomenclature

## Inputs

- Data frame of metabolite names (e.g. from getmwstudies(studyid, typedata) for a Metabolomics Workbench study such as ST000915)
- Metabolite names and concentration with metadata information (MetENP expected input)

## Outputs

- Annotated data frame / refmet_annotations.csv with per-metabolite RefMet main_class, sub_class, and super_class fields
- RefMet standard-nomenclature mappings for each input metabolite name

## How to apply

First load the MetENP R package, explicitly attaching KEGGREST and the other Imports dependencies (e.g. plyr, dplyr, jsonlite, httr) since Imports packages are not auto-attached. Retrieve the metabolite-name data frame for the target study with getmwstudies(studyid, typedata) — for example study ST000915 — then pass that data frame to convert_refmet(), which queries RefMet's standardized nomenclature to map each metabolite name to its RefMet identifiers and hierarchical class categories. The result carries main_class, sub_class, and super_class columns for every metabolite row; write the annotated table out (e.g. refmet_annotations.csv) so it can feed the metclassenrichment step of the MetENP pipeline. The rationale is that RefMet provides standard nomenclature enabling efficient comparisons of metabolite data across experiments, and MetENP's enrichment scoring operates on these standardized classes rather than raw metabolite names.

## Related tools

- **MetENP** (Provides the convert_refmet() function that performs the RefMet class mapping; loaded via devtools::install("MetENP") or sourced as path/convert_refmet.R) — https://github.com/metabolomicsworkbench/MetENP
- **RefMet** (External reference resource supplying the standardized metabolite nomenclature and the main_class/sub_class/super_class hierarchy that convert_refmet maps names to)
- **KEGGREST** (Bioconductor dependency of MetENP that must be explicitly library()-attached before running convert_refmet because Imports packages are not attached automatically) — https://github.com/Bioconductor/KEGGREST

## Examples

```
library(KEGGREST); metdata <- getmwstudies("ST000915", "metabolites"); refmet_annot <- convert_refmet(metdata); write.csv(refmet_annot, "refmet_annotations.csv")
```

## Evaluation signals

- Every input metabolite row in refmet_annotations.csv carries non-missing main_class, sub_class, and super_class fields (complete annotation coverage)
- The output row count equals the input metabolite-name data frame row count (no metabolites dropped during mapping)
- Class values conform to the RefMet hierarchical vocabulary (main_class/sub_class/super_class triplets), not free-text names
- Downstream metclassenrichment runs without name-matching failures, indicating names were standardized before enrichment

## Limitations

- Mapping quality depends on input metabolite names being resolvable to RefMet standard nomenclature; unresolvable names leave class fields unpopulated
- MetENP's Imports dependencies (e.g. KEGGREST) are not attached on library(MetENP) and must be loaded explicitly, a common failure mode
- No software version number is stated for MetENP in the article, so reproducibility of the RefMet mapping behavior requires pinning the GitHub checkout

## Evidence

- [methods] convert_refmet assigns RefMet class categories: [evidence span withheld — anchor status: not_found]
- [intro] RefMet provides standard nomenclature: "RefMet (A Reference list of Metabolite names (10), that provides standard nomenclature for metabolites, enabling efficient comparisons between metabolite data across experiments"
- [discussion] map metabolites to standardized metabolite classes: "a) map metabolites to standardized metabolite classes"
- [readme] convert_refmet.R source line: "source('path/convert_refmet.R')"
- [readme] MetENP dependencies note: "MetENP package depends on following Bioconductor packages to function properly: KEGGREST, KEGGgraph, and pathview."
