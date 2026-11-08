---
name: lipidomics-feature-table-prefiltering
description: Use when raw vendor lipidomics identification/quantification tables (LipidSearch
  or LIQUID outputs) or Metabolomics Workbench experiments must be imported, pre-filtered,
  and lipid species decomposed into class/chain/double-bond strata.
when_to_use_negative:
- Input is already a parsed feature table with lipid class/chain/double-bond annotations
  — proceed directly to normalization or differential analysis instead
- Data are not lipid identification/quantification outputs from LipidSearch, LIQUID,
  or the Metabolomics Workbench (ADViSELipidomics only copes with these sources)
- The goal is normalization to absolute concentrations via internal standards — that
  is a downstream step that assumes the import/parsing stage is already done
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: ADViSELipidomics
  role: Shiny app performing the data import, feature pre-filtering, and LIPID MAPS-based
    parsing of lipid species
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: Shiny
  role: Framework in which the ADViSELipidomics data-import layer is implemented
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose identification/quantification output files are the primary
    import format
- name: LIQUID
  role: Alternative lipid identification/quantification tool whose outputs ADViSELipidomics
    can import
- name: metabolomicsWorkbenchR
  role: R package whose do_query() function downloads Metabolomics Workbench experiments
    as an alternative input source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:a18c2f7ffaa6e2d134aa819ba4c1961e5b108f07b614250add22530f64dac19e
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Lipidomics feature-table pre-filtering and lipid species parsing

## Summary

Data-import stage of ADViSELipidomics in which lipid identification/quantification outputs (e.g. LipidSearch files) are loaded, pre-filtered to retain features of interest, and each lipid species identifier is parsed using LIPID MAPS classification into lipid class, chain length and double-bond count.

## When to use

Use when raw vendor lipidomics identification/quantification tables (LipidSearch or LIQUID outputs) or Metabolomics Workbench experiments must be imported, pre-filtered, and lipid species decomposed into class/chain/double-bond strata.

## When NOT to use

- Input is already a parsed feature table with lipid class/chain/double-bond annotations — proceed directly to normalization or differential analysis instead
- Data are not lipid identification/quantification outputs from LipidSearch, LIQUID, or the Metabolomics Workbench (ADViSELipidomics only copes with these sources)
- The goal is normalization to absolute concentrations via internal standards — that is a downstream step that assumes the import/parsing stage is already done

## Inputs

- LipidSearch identification and quantification output files (as in Case Study 1)
- LIQUID output files for lipid identification and quantification
- Accompanying target/sample-information file
- Metabolomics Workbench lipidomic experiment (fetched via do_query() from metabolomicsWorkbenchR)
- Case_Study_1.zip example data from the ShinyFabio/ADViSELipidomics_book data_example repository

## Outputs

- Pre-filtered feature matrix (non-informative features removed)
- Parsed lipid-species table with one row per lipid species and columns for lipid class, chain length and double bonds (e.g. parsed_lipid_species.csv)

## How to apply

Load the identification and quantification files together with the accompanying target file into the data-import layer of the ADViSELipidomics Shiny app (launched via library(ADViSELipidomics); run_ADViSELipidomics()). Apply the import-stage pre-filtering to remove non-informative entries from the feature matrix before any downstream exploratory or statistical analysis. Then parse each lipid species identifier using the LIPID MAPS classification embedded in ADViSELipidomics, splitting each species into lipid class, chain length and double-bond count as separate strata. For Metabolomics Workbench experiments, retrieve the data in real time with the do_query() function from the metabolomicsWorkbenchR package instead of local files. The rationale is that downstream steps (normalization with internal lipid standards, differential abundance testing, batch effect correction) operate on the parsed, filtered feature matrix, so mis-parsed species propagate through the whole workflow.

## Related tools

- **ADViSELipidomics** (Shiny app performing the data import, feature pre-filtering, and LIPID MAPS-based parsing of lipid species) — https://github.com/ShinyFabio/ADViSELipidomics
- **Shiny** (Framework in which the ADViSELipidomics data-import layer is implemented) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose identification/quantification output files are the primary import format)
- **LIQUID** (Alternative lipid identification/quantification tool whose outputs ADViSELipidomics can import)
- **metabolomicsWorkbenchR** (R package whose do_query() function downloads Metabolomics Workbench experiments as an alternative input source)

## Examples

```
devtools::install_github("ShinyFabio/ADViSELipidomics"); library(ADViSELipidomics); run_ADViSELipidomics()
```

## Evaluation signals

- parsed_lipid_species.csv exists and is valid CSV with class/chain-length/double-bond columns
- Every lipid species present in the input identification file appears in the parsed table with all three strata populated
- No lipid species failed LIPID MAPS classification (unparseable identifiers are absent)
- The pre-filtered feature matrix contains only features of interest — non-informative entries from the raw quantification file are removed
- Parsed class/chain/double-bond values agree with the LIPID MAPS nomenclature of the original species strings

## Limitations

- Only handles outputs from LipidSearch and LIQUID and data from the Metabolomics Workbench; other vendor formats are out of scope
- Retrieving Metabolomics Workbench experiments in real time requires an active internet connection
- Batch-correction method used downstream is not named in the paper, and QC replicate samples show very low variance, limiting power to detect non-linear drift

## Evidence

- [readme] It handles the outputs from LipidSearch and LIQUID for lipid identification and quantification, and with data available from the Metabolomics Workbench.: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [abstract] Parsing via LIPID MAPS classification is the core import capability.: "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [methods] App launch entry point for the import layer.: "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [readme] Alternative real-time data source for the import stage.: "ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository using the `do_query()` function"
- [supplementary] Example data bundle for testing the import with LipidSearch output.: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
