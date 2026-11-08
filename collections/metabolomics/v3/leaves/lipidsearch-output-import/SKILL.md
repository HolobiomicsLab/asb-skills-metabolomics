---
name: lipidsearch-output-import
description: Use when the input consists of LipidSearch output files (identification
  + quantification, with an accompanying target file) that must be loaded into ADViSELipidomics,
  pre-filtered, and parsed into lipid class/chain/double-bond strata.
when_to_use_negative:
- Input data comes from LIQUID rather than LipidSearch — ADViSELipidomics handles
  both, but this skill's import/parsing specifics target LipidSearch output files
- Data are already a pre-filtered, parsed feature table (e.g. previously exported
  from ADViSELipidomics) — re-importing would duplicate preprocessing
- The experiment is a Metabolomics Workbench lipidomic dataset — use the do_query()
  download path from metabolomicsWorkbenchR instead of file-based LipidSearch import
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ADViSELipidomics
  role: Shiny app whose data-import layer loads LipidSearch outputs, applies feature
    pre-filtering, and parses lipid species using LIPID MAPS classification
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Upstream tool producing the identification and quantification output files
    consumed by this import step
- name: Shiny
  role: Web framework in which the ADViSELipidomics data-import interface is implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:9885d4fe0aa03c50340094cdd2815f0cea71cd300a5a01c825346314eabd8b0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipidsearch-output-import

## Summary

Import LipidSearch identification and quantification output files (plus a target file) into the ADViSELipidomics Shiny app, pre-filter features of interest, and parse lipid species identifiers using LIPID MAPS classification to stratify class, chain length, and double-bond count.

## When to use

Use when the input consists of LipidSearch output files (identification + quantification, with an accompanying target file) that must be loaded into ADViSELipidomics, pre-filtered, and parsed into lipid class/chain/double-bond strata.

## When NOT to use

- Input data comes from LIQUID rather than LipidSearch — ADViSELipidomics handles both, but this skill's import/parsing specifics target LipidSearch output files
- Data are already a pre-filtered, parsed feature table (e.g. previously exported from ADViSELipidomics) — re-importing would duplicate preprocessing
- The experiment is a Metabolomics Workbench lipidomic dataset — use the do_query() download path from metabolomicsWorkbenchR instead of file-based LipidSearch import

## Inputs

- LipidSearch identification output file
- LipidSearch quantification output file
- Accompanying target file describing samples
- Case_Study_1.zip example data (ShinyFabio/ADViSELipidomics_book data_example) for validation runs

## Outputs

- Pre-filtered feature matrix (non-informative entries removed)
- Parsed lipid species table with one row per lipid species and columns for class, chain length, and double bonds

## How to apply

Install ADViSELipidomics (R >= 4.0) via devtools::install_github("ShinyFabio/ADViSELipidomics") and launch it with library(ADViSELipidomics); run_ADViSELipidomics(). Load the LipidSearch identification and quantification files together with the target file into the data-import layer of the Shiny app (for testing, the Case_Study_1.zip files from the ShinyFabio/ADViSELipidomics_book data_example repository reproduce the paper's inputs). Apply the import-stage pre-filtering to remove non-informative entries from the feature matrix, then parse each lipid species identifier using the embedded LIPID MAPS classification to extract lipid class, chain length, and double-bond count as separate strata. The rationale is that downstream exploratory and statistical analyses in ADViSELipidomics rely on these parsed stratified characteristics per lipid species.

## Related tools

- **ADViSELipidomics** (Shiny app whose data-import layer loads LipidSearch outputs, applies feature pre-filtering, and parses lipid species using LIPID MAPS classification) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Upstream tool producing the identification and quantification output files consumed by this import step)
- **Shiny** (Web framework in which the ADViSELipidomics data-import interface is implemented)

## Examples

```
library(ADViSELipidomics); run_ADViSELipidomics()
```

## Evaluation signals

- parsed_lipid_species.csv exists and is valid CSV
- The parsed table contains class, chain length, and double-bond fields for every lipid species present in the input files
- No lipid species failed LIPID MAPS classification during parsing
- The pre-filtered feature matrix excludes non-informative entries while retaining features of interest
- Case Study 1 example files load without errors into the data-import layer, reproducing the paper's workflow

## Limitations

- The Shiny interface layout depends on screen size and monitor resolution and may deviate from the intended design; users may need to adjust zoom (Ctrl +/- or Command +/-)
- Installation requires OS-specific supplementary dependencies (Rtools on Windows; brew imagemagick@6 and cairo on macOS; several apt packages on Ubuntu 18.04) and can be slow if many packages are compiled
- Chrome is the tested browser for the Docker version; Firefox can have problems

## Evidence

- [abstract] Data import stage imports, pre-filters, and parses lipids: "It handles the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [intro] LIPID MAPS parsing of lipid species: "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [methods] Installation command: "devtools::install_github("ShinyFabio/ADViSELipidomics")"
- [readme] Launch command: "library("ADViSELipidomics")
run_ADViSELipidomics()"
- [readme] Case Study 1 example data location: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
- [supplementary] Case Study 1 download: "https://github.com/ShinyFabio/ADViSELipidomics_book/raw/main/data_example/Case_Study_1.zip/"
