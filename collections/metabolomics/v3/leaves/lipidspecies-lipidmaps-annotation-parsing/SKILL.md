---
name: lipidspecies-lipidmaps-annotation-parsing
description: Use when lipid quantification matrices arrive as LipidSearch or LIQUID
  vendor outputs (or a Metabolomics Workbench experiment download) and lipid species
  names must be decomposed into LIPID MAPS-compliant structural information before
  analysis.
when_to_use_negative:
- Input lipid identifiers do not follow LIPID MAPS classification conventions or come
  from identification software other than LipidSearch or LIQUID
- Data are already a parsed, feature-annotated matrix — re-parsing vendor outputs
  would be redundant
- The goal is Metabolomics Workbench retrieval or normalization itself rather than
  structuring the lipid species identifiers
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3820
tools:
- name: ADViSELipidomics
  role: Shiny app whose preprocessing module parses lipid species identifiers using
    LIPID MAPS classification from vendor outputs
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of the lipid identification and quantification output files that the
    parser ingests (Case Study 1 files)
- name: LIQUID
  role: Alternative lipid identification/quantification software whose outputs the
    parser can ingest
- name: Metabolomics Workbench
  role: Online repository from which suitable selected lipidomic experiments can be
    downloaded in real time for parsing
- name: metabolomicsWorkbenchR
  role: R package whose do_query() function fetches Metabolomics Workbench experiments
    into the app
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:e15caa337903447871f6f1cd0995182072b78dd406d626ba3748e6302555116d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIQUID
  - LipidSearch
  - Metabolomics Workbench
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3820
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lipidspecies-lipidmaps-annotation-parsing

## Summary

Parses lipid species identifiers from LipidSearch/LIQUID or Metabolomics Workbench outputs using the LIPID MAPS classification, so each species' class and chain information can be attached to sample metadata for downstream normalization and differential analysis in ADViSELipidomics.

## When to use

Use when lipid quantification matrices arrive as LipidSearch or LIQUID vendor outputs (or a Metabolomics Workbench experiment download) and lipid species names must be decomposed into LIPID MAPS-compliant structural information before analysis.

## When NOT to use

- Input lipid identifiers do not follow LIPID MAPS classification conventions or come from identification software other than LipidSearch or LIQUID
- Data are already a parsed, feature-annotated matrix — re-parsing vendor outputs would be redundant
- The goal is Metabolomics Workbench retrieval or normalization itself rather than structuring the lipid species identifiers

## Inputs

- LipidSearch output files for lipid identification and quantification (e.g. those in Case_Study_1.zip from the ADViSELipidomics_book data_example repository)
- LIQUID output files for lipid identification and quantification
- A lipidomic experiment downloaded in real time from the Metabolomics Workbench via metabolomicsWorkbenchR::do_query()
- Sample metadata/information available on the samples in the experiment

## Outputs

- Parsed data matrix of lipid species annotated per the LIPID MAPS classification, aligned with sample information
- Structured species information suitable for exploratory and statistical analyses and internal-standard-based normalization to absolute concentration values per lipid and sample

## How to apply

Load the vendor output files (e.g. the LipidSearch outputs shipped in Case_Study_1.zip) into the ADViSELipidomics preprocessing module of the Shiny app, which is launched via library(ADViSELipidomics); run_ADViSELipidomics() under R > 4.0. The app handles outputs from LipidSearch and LIQUID as well as experiments fetched in real time from the Metabolomics Workbench via do_query() from the {metabolomicsWorkbenchR} package. Parsing extracts information from each lipid species name according to the LIPID MAPS classification and joins it with sample-level information available in the experiment. The rationale is that lipid identifiers such as lipid class and chain composition must be structured before exploratory analysis, internal-standard-based normalization to absolute concentrations, or differential abundance testing in simple and complex designs can proceed. The parsed matrix can be validated against the article's Case Study 1 supplementary results, which report the normalized concentration values per lipids and samples derived from these files.

## Related tools

- **ADViSELipidomics** (Shiny app whose preprocessing module parses lipid species identifiers using LIPID MAPS classification from vendor outputs) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of the lipid identification and quantification output files that the parser ingests (Case Study 1 files))
- **LIQUID** (Alternative lipid identification/quantification software whose outputs the parser can ingest)
- **Metabolomics Workbench** (Online repository from which suitable selected lipidomic experiments can be downloaded in real time for parsing)
- **metabolomicsWorkbenchR** (R package whose do_query() function fetches Metabolomics Workbench experiments into the app)

## Examples

```
devtools::install_github("ShinyFabio/ADViSELipidomics"); library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Every parsed lipid species name resolves consistently against the LIPID MAPS classification (class/species level information extracted without malformed identifiers)
- The parsed matrix row/column counts match the input LipidSearch/LIQUID output (all lipids and samples retained after parsing)
- Downstream internal-standard-normalized concentrations per lipids and samples reproduce Case Study 1 values reported in the article's Supplementary Material
- Parsed species align with sample metadata so the monocytes-vs-iDCs two-group comparison of Case Study 1 can be executed
- Metabolomics Workbench data path yields the same parse structure as vendor-output data for suitable selected experiments

## Limitations

- Requires R > 4.0 and OS-specific dependencies (Rtools on Windows; brew imagemagick@6 and cairo on macOS; several apt packages on Ubuntu) before installation
- The app interface can render differently depending on monitor size/resolution, requiring zoom adjustments (Ctrl/Cmd +/−)
- Docker version was tested on Chrome; Firefox may have problems (http://localhost:80/)
- Batch correction method is applied but the article discussion does not identify the method name, limiting reproducibility of that downstream step

## Evidence

- [abstract] Parsing abstracts/vendor outputs: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses"
- [readme] Supported input software: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification, and with data available from the Metabolomics Workbench."
- [abstract] Normalization output depends on parsing: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [supplementary] Case Study 1 input files: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
- [supplementary] Metabolomics Workbench retrieval path: "ADViSELipidomics can download in real-time suitable selected lipidomic experiments from the online repository using the `do_query()` function from the `{metabolomicsWorkbenchR}` package"
- [methods] Launch invocation: "library("ADViSELipidomics")
run_ADViSELipidomics()"
