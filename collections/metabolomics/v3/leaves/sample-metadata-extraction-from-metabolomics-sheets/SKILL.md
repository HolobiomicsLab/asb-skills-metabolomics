---
name: sample-metadata-extraction-from-metabolomics-sheets
description: Use when the input is a platform vendor metabolomics workbook or export
  (Excel sheets such as Metabolon 'OrigScale', Nightingale v1) and the sample annotation
  (batch, sample_id, run_day) must be separated from raw abundance data for downstream
  QC.
when_to_use_negative:
- The data are already three separate plain tables (data/samples/features) such as
  dummy_data.csv, dummy_samples.csv, dummy_features.csv — no sheet parsing is needed
- The goal is downstream QC filtering (sample/feature missingness, total_peak_area_sd,
  PCA outliers) rather than extracting metadata from a vendor worksheet
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: metaboprep
  role: Provides the platform-specific importers (read_metabolon, read_nightingale)
    that pull sample metadata, abundance data, and feature metadata out of preformed
    worksheets
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which metaboprep is run and the extracted components
    are exported as tab-delimited files
- name: readxl
  role: Underlying Excel-reading dependency used by read_metabolon to parse the source
    workbook sheets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:045d271ab35477b6a6e81eeccb465d143ae9907d1ded5cfe9cd1dfe22fbe8949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  - readxl
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sample-metadata-extraction-from-metabolomics-sheets

## Summary

Extracts the sample metadata table from platform-specific metabolomics source files (e.g. Nightingale or Metabolon Excel workbooks) alongside the metabolite abundance matrix and metabolite metadata, as the first step of the metaboprep preanalysis workflow.

## When to use

Use when the input is a platform vendor metabolomics workbook or export (Excel sheets such as Metabolon 'OrigScale', Nightingale v1) and the sample annotation (batch, sample_id, run_day) must be separated from raw abundance data for downstream QC.

## When NOT to use

- The data are already three separate plain tables (data/samples/features) such as dummy_data.csv, dummy_samples.csv, dummy_features.csv — no sheet parsing is needed
- The goal is downstream QC filtering (sample/feature missingness, total_peak_area_sd, PCA outliers) rather than extracting metadata from a vendor worksheet

## Inputs

- Metabolon Excel workbook (e.g. metabolon_v1.1_example.xlsx) read from sheet 'OrigScale'
- Nightingale Health Excel workbook (e.g. nightingale_v1_example.xlsx)
- Preformed worksheets containing (semi-)quantified metabolite data, sample metadata, and metabolite metadata

## Outputs

- Raw metabolite abundance matrix (tab-delimited text file)
- Sample metadata table (technical batch information, sample identifiers such as sample_id, run_day, box_id, lot)
- Metabolite/feature metadata table (e.g. metabolite_id, platform, pathway, kegg, group_hmdb)

## How to apply

Load the metaboprep package in R and locate a packaged example source file (e.g. metabolon_v1.1_example.xlsx) via system.file('extdata', ...). Call the platform-specific importer — read_metabolon with sheet='OrigScale' for Metabolon Excel files, or read_nightingale for Nightingale Health Excel files — with return_Metaboprep=FALSE so the raw components are returned as a list rather than a Metaboprep object. Inspect the returned structure (str(dat)) to confirm the abundance matrix, sample table, and feature table were parsed, noting platform-specific annotation columns such as run_day, box_id, lot, and xenobiotic super_pathway membership. Write the unaltered abundance matrix, sample metadata, and metabolite metadata to disk as tab-delimited text files (write.table with sep='\t', quote=FALSE), which is the package's stated goal of 'saving datasets in tab-delimited format for use elsewhere'. Validate by reading each file back with read.delim and confirming row/column counts match the in-memory objects.

## Related tools

- **metaboprep** (Provides the platform-specific importers (read_metabolon, read_nightingale) that pull sample metadata, abundance data, and feature metadata out of preformed worksheets) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which metaboprep is run and the extracted components are exported as tab-delimited files)
- **readxl** (Underlying Excel-reading dependency used by read_metabolon to parse the source workbook sheets)

## Examples

```
mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"), sheet = "OrigScale", return_Metaboprep = FALSE)
```

## Evaluation signals

- str(dat) shows three parsed components: an abundance matrix, a sample table, and a feature table with expected annotation columns
- Each written .txt file, when re-imported with read.delim, reproduces the exact dimensions of the in-memory abundance matrix, sample table, and feature table with no rows or columns dropped or altered
- Sample metadata retains platform-specific columns (e.g. sample_id, run_day, box_id, lot for Metabolon; super_pathway flags for Nightingale)
- Files are tab-delimited with quote=FALSE so they can be consumed by the Metaboprep(data=, features=, samples=) constructor downstream

## Limitations

- The importer is platform-specific: read_metabolon must be pointed at the correct sheet name ('OrigScale'); using the wrong sheet yields different content
- Excel workbooks are the documented extraction source for this step; other platforms (Olink .txt, SomaLogic .adat, generic CSV) are packaged as examples but the raw-extraction path here targets preformed worksheets
- With return_Metaboprep=FALSE the importer returns a list, not a Metaboprep object, so a separate Metaboprep(...) constructor call is needed before QC

## Evidence

- [abstract] The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites"
- [intro] Read in and process (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere: "Read in and processes (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
- [methods] read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"): "read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep")"
- [readme] sheet             = "OrigScale", ## The name of the sheet in the excel file to read in: "sheet             = "OrigScale", ## The name of the sheet in the excel file to read in"
