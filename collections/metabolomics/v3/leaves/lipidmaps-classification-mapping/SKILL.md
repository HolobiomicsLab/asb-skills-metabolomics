---
name: lipidmaps-classification-mapping
description: Use when importing lipid identification/quantification tables (LipidSearch
  or LIQUID outputs, or Metabolomics Workbench experiments) and each lipid species
  name must be decomposed into class, chain length, and double-bond strata.
when_to_use_negative:
- Input is already a parsed feature table with separate class/chain/double-bond columns
  — re-parsing is unnecessary
- Lipid species names do not follow LIPID MAPS nomenclature, so classification would
  fail for many entries
- The goal is analysis of non-lipidomics data (e.g., metabolomics beyond the lipid
  species ADViSELipidomics handles)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny app whose data-import layer pre-filters features and parses lipid species
    using LIPID MAPS classification into class, chain length, and double bonds
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: R web framework in which the ADViSELipidomics import and parsing layer is
    implemented
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of lipid identification and quantification output files whose species
    names are parsed via LIPID MAPS classification
- name: LIQUID
  role: Alternative source of lipid identification/quantification outputs handled
    by the same parsing step
- name: metabolomicsWorkbenchR
  role: Provides do_query() to fetch Metabolomics Workbench lipidomic experiments
    as alternative parsing input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:c0b744b3720d3b136be8a2a9b78f790db119c2230b6ddbb41d73d9838b75dd50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIQUID
  - LipidSearch
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# lipidmaps-classification-mapping

## Summary

Parses lipid species identifiers from LipidSearch/LIQUID outputs using the LIPID MAPS classification, stratifying each species into lipid class, chain length, and double-bond count for downstream exploratory and statistical analyses in ADViSELipidomics.

## When to use

Use when importing lipid identification/quantification tables (LipidSearch or LIQUID outputs, or Metabolomics Workbench experiments) and each lipid species name must be decomposed into class, chain length, and double-bond strata.

## When NOT to use

- Input is already a parsed feature table with separate class/chain/double-bond columns — re-parsing is unnecessary
- Lipid species names do not follow LIPID MAPS nomenclature, so classification would fail for many entries
- The goal is analysis of non-lipidomics data (e.g., metabolomics beyond the lipid species ADViSELipidomics handles)

## Inputs

- LipidSearch identification and quantification output files (e.g., Case_Study_1.zip example data)
- LIQUID output files (alternative identification/quantification source)
- Metabolomics Workbench lipidomic experiment downloaded via do_query() from metabolomicsWorkbenchR
- Accompanying target/sample metadata file

## Outputs

- Parsed lipid species table with one row per lipid species and columns for lipid class, chain length, and double-bond count (e.g., parsed_lipid_species.csv)
- Pre-filtered feature matrix used as the basis for parsing

## How to apply

Load the identification and quantification files (e.g., LipidSearch output plus the accompanying target file) into the data-import layer of the ADViSELipidomics Shiny app, first applying the import-stage pre-filtering to remove non-informative features. Then parse each lipid species identifier using the LIPID MAPS classification embedded in ADViSELipidomics, splitting the species name into three separate strata: lipid class, chain length, and number of double bonds. The rationale is that these parsed attributes, together with sample metadata, enable the app's exploratory and statistical analyses (class-level summaries, differential abundance testing). Write the parsed result as one row per lipid species with class/chain-length/double-bond columns, and verify that every input species was successfully classified under LIPID MAPS.

## Related tools

- **ADViSELipidomics** (Shiny app whose data-import layer pre-filters features and parses lipid species using LIPID MAPS classification into class, chain length, and double bonds) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (R web framework in which the ADViSELipidomics import and parsing layer is implemented) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of lipid identification and quantification output files whose species names are parsed via LIPID MAPS classification)
- **LIQUID** (Alternative source of lipid identification/quantification outputs handled by the same parsing step)
- **metabolomicsWorkbenchR** (Provides do_query() to fetch Metabolomics Workbench lipidomic experiments as alternative parsing input)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- parsed_lipid_species.csv exists and is valid CSV
- The parsed table contains lipid class, chain length, and double-bond fields for every lipid species in the input
- No lipid species failed LIPID MAPS classification (zero unparseable identifiers)
- Row count of the parsed table matches the number of species surviving the import-stage pre-filtering
- Downstream stratified analyses (e.g., class-level summaries) can consume the parsed columns without further restructuring

## Limitations

- Parsing relies on LIPID MAPS classification, so non-conforming lipid species names cannot be stratified
- The Shiny interface rendering depends on monitor resolution/zoom, which can complicate interactive use of the import layer
- When installing many packages from source, installation can take a long time depending on hardware and OS

## Evidence

- [other] ADViSELipidomics imports the data, pre-filters them based on features of interest, and parses lipids to stratify their characteristics into class, chain length and double bonds.: [evidence span withheld — anchor status: not_found]
- [abstract] extracts information by parsing lipid species (using LIPID MAPS classification): "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [readme] It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [readme] ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository using the `do_query()` function: "ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository using the `do_query()` function"
