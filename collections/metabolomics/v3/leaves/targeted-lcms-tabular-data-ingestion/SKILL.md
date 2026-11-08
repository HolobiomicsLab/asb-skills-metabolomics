---
name: targeted-lcms-tabular-data-ingestion
description: Use when starting from a vendor peak table exported from tools like SCIEX
  OS (dMRM polarity-switching MRM runs) and a tab-delimited text file of peak areas
  must be loaded, checked for missing/incorrect mandatory columns, and turned into
  a SummarizedExperiment for mzQuality QC workflows.
when_to_use_negative:
- The data is not a peak-integrated targeted MRM/LC-MS table but raw vendor spectra
  files (mzML, NetCDF) that still need feature detection and peak integration — mzQuality
  expects peak-integrated data as input
- The compounds are untargeted/unknown features without assigned internal standards
  — mzQuality's mandatory-column checks and IS-based calculations assume known metabolites
  with corresponding stable isotope-labeled IS
- The tab-delimited file lacks the mandatory columns mzQuality checks for (e.g. missing
  sample type or IS assignment), and they cannot be supplied — readData will fail
  on such files
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzQuality
  role: Provides the readData and buildExperiment functions used to ingest and validate
    the tab-delimited peak table and convert it into a SummarizedExperiment
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment for mzQuality (version 4.0 or later required) in which
    the ingestion functions are executed
  repo: https://github.com/hankemeierlab/mzQuality
- name: SCIEX OS
  role: Upstream vendor software that exports the peak-integrated data file that readData
    consumes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:fdccca477272cccdc104381f9007795558bd8d46fe5ac1a567e777871f457ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - SCIEX OS
  - mzQuality
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# targeted-lcms-tabular-data-ingestion

## Summary

Ingests peak-integrated targeted LC-MS results as a tab-delimited file (with mandatory columns such as compound areas, internal standard assignments, sample type, and batch), validates the required columns, and converts them into a SummarizedExperiment object that downstream mzQuality calculations use.

## When to use

Use when starting from a vendor peak table exported from tools like SCIEX OS (dMRM polarity-switching MRM runs) and a tab-delimited text file of peak areas must be loaded, checked for missing/incorrect mandatory columns, and turned into a SummarizedExperiment for mzQuality QC workflows.

## When NOT to use

- The data is not a peak-integrated targeted MRM/LC-MS table but raw vendor spectra files (mzML, NetCDF) that still need feature detection and peak integration — mzQuality expects peak-integrated data as input
- The compounds are untargeted/unknown features without assigned internal standards — mzQuality's mandatory-column checks and IS-based calculations assume known metabolites with corresponding stable isotope-labeled IS
- The tab-delimited file lacks the mandatory columns mzQuality checks for (e.g. missing sample type or IS assignment), and they cannot be supplied — readData will fail on such files

## Inputs

- Tab-delimited peak-integrated data table (e.g. .tsv) with mandatory columns for compound peak areas, internal standard assignments, sample type, and batch
- Vendor peak table from SCIEX OS (version 2.1.6.59781) or similar targeted MRM acquisition software
- SummarizedExperiment object (alternative accepted input)

## Outputs

- Validated data frame (combined) with all mandatory columns present
- SummarizedExperiment object (exp) suitable for mzQuality analysis functions

## How to apply

Point the readData function at a tab-delimited file (e.g. system.file("extdata", "example.tsv", package = "mzQuality")) containing peak-integrated data for compounds and their stable isotope-labeled internal standards, sample type, and batch columns; readData checks whether all mandatory columns are present and whether the data is in the correct format. Pass the validated data frame to buildExperiment, which converts it into a SummarizedExperiment that mzQuality uses for all calculations and analyses. The example packaged with mzQuality contains 105 compounds and 584 samples and can be loaded the same way to explore the workflow. Subsequent steps such as doAnalysis rely on this SummarizedExperiment, so ingestion errors (missing mandatory columns, wrong file format) will propagate; correct them at this stage rather than downstream.

## Related tools

- **mzQuality** (Provides the readData and buildExperiment functions used to ingest and validate the tab-delimited peak table and convert it into a SummarizedExperiment) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment for mzQuality (version 4.0 or later required) in which the ingestion functions are executed) — https://github.com/hankemeierlab/mzQuality
- **SCIEX OS** (Upstream vendor software that exports the peak-integrated data file that readData consumes)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path))
```

## Evaluation signals

- readData completes without raising errors about missing or incorrectly formatted mandatory columns
- buildExperiment returns a SummarizedExperiment whose assay dimensions match the number of compounds and samples in the input file
- The experiment object can be passed to doAnalysis without modification and produces expected slots (assay, rowData, colData)
- The packaged example (105 compounds × 584 samples) loads reproducibly via system.file("extdata", "example.tsv", package = "mzQuality")

## Limitations

- Data must be peak-integrated beforehand (e.g. by SCIEX OS dMRM integration); mzQuality does not itself process raw peak data
- Any missing value (NA) is treated as an absent data point and will not be used for calculations, so ingestion does not impute or flag missing entries
- readData assumes a tab-delimited text file (or a SummarizedExperiment); other formats must be converted first

## Evidence

- [methods] readData checks mandatory columns and format: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [methods] buildExperiment creates the SummarizedExperiment: "The `buildExperiment` function will then take the data and create an experiment object that can be used for analysis."
- [readme] SummarizedExperiment is the core mzQuality object: [evidence span withheld — anchor status: external_doc]
- [abstract] Data must be peak-integrated: "It processes peak-integrated data independently of vendor software"
- [readme] readData from the README: [evidence span withheld — anchor status: external_doc]
- [methods] Vendor integration via SCIEX OS: "The data were integrated using SCIEX OS (version 2.1.6.59781)"
- [methods] doAnalysis depends on SummarizedExperiment: "This is automatically done when the `doAnalysis` function is called."
