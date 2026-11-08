---
name: excel-metabolomics-file-parsing-with-readxl
description: Use when the input is a vendor Excel metabolomics workbook (e.g. metabolon_v1.1_example.xlsx
  with sheet='OrigScale', or nightingale_v1_example.xlsx) and you need the raw abundance
  matrix plus sample and feature metadata as separate objects.
when_to_use_negative:
- The source data is already a tab-delimited or CSV feature table (e.g. olink_v1_example.txt
  or dummy_data.csv with dummy_samples.csv/dummy_features.csv) — the platform-specific
  Excel importers are not needed.
- The goal is quality control, filtering, or report generation on an already-parsed
  dataset — use metaboprep's quality_control pipeline instead of the raw extraction
  step.
- A ready-made Metaboprep object is the desired end state — set return_Metaboprep=TRUE
  rather than extracting the raw data/samples/features list.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metaboprep
  role: Provides the platform-specific importer functions read_nightingale and read_metabolon
    that parse the Excel workbooks into data, samples, and features components
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep importers are called and the extracted
    components are written to disk via write.table
  repo: https://github.com/MRCIEU/metaboprep
- name: readxl
  role: Underlying package metaboprep uses to read the .xlsx Excel sheets (e.g. Metabolon
    'OrigScale') into R data frames
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:70d2ed7eabf1557fae5f254650e90b5d4a2d6cb2ce187898728982e13b9270be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  - readxl
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# excel-metabolomics-file-parsing-with-readxl

## Summary

Extracts the abundance matrix, sample metadata, and metabolite (feature) metadata from platform-specific metabolomics Excel files (e.g. Nightingale Health, Metabolon) using metaboprep's importer functions built on readxl, yielding components that can be exported as tab-delimited datasets.

## When to use

Use when the input is a vendor Excel metabolomics workbook (e.g. metabolon_v1.1_example.xlsx with sheet='OrigScale', or nightingale_v1_example.xlsx) and you need the raw abundance matrix plus sample and feature metadata as separate objects.

## When NOT to use

- The source data is already a tab-delimited or CSV feature table (e.g. olink_v1_example.txt or dummy_data.csv with dummy_samples.csv/dummy_features.csv) — the platform-specific Excel importers are not needed.
- The goal is quality control, filtering, or report generation on an already-parsed dataset — use metaboprep's quality_control pipeline instead of the raw extraction step.
- A ready-made Metaboprep object is the desired end state — set return_Metaboprep=TRUE rather than extracting the raw data/samples/features list.

## Inputs

- Metabolon Excel workbook (.xlsx, data sheet 'OrigScale') such as metabolon_v1.1_example.xlsx or metabolon_v1.2_example.xlsx
- Nightingale Health Excel workbook such as nightingale_v1_example.xlsx
- R environment with metaboprep and readxl installed

## Outputs

- Raw abundance matrix (data component) as a separate in-memory object
- Sample metadata table (e.g. technical batch information, sample identifiers)
- Metabolite/feature metadata table (e.g. super_pathway membership, derived-measure flags)
- Tab-delimited text files of the abundance matrix, sample metadata, and metabolite metadata

## How to apply

Locate the packaged example file with system.file('extdata', ..., package='metaboprep'), then call the platform-specific importer — read_nightingale for Nightingale Health Excel files or read_metabolon for Metabolon files with sheet='OrigScale' — with return_Metaboprep=FALSE so the function returns a list of raw data, samples, and features components rather than a Metaboprep object. Inspect the structure of each component (e.g. str(dat)) to confirm the abundance matrix parsed correctly and to record platform-specific annotation columns such as xenobiotic super_pathway membership or derived-measure flags. Because this skill covers raw extraction (not QC), write the unaltered abundance matrix, sample table, and feature table to disk with write.table(sep='\t', quote=FALSE), then re-import with read.delim and verify row/column counts match the in-memory objects. This corresponds to the package's stated first goal: reading in and processing (un)targeted metabolite data and saving the datasets in tab-delimited format for use elsewhere.

## Related tools

- **metaboprep** (Provides the platform-specific importer functions read_nightingale and read_metabolon that parse the Excel workbooks into data, samples, and features components) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep importers are called and the extracted components are written to disk via write.table) — https://github.com/MRCIEU/metaboprep
- **readxl** (Underlying package metaboprep uses to read the .xlsx Excel sheets (e.g. Metabolon 'OrigScale') into R data frames)

## Examples

```
read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"), sheet = "OrigScale", return_Metaboprep = FALSE)
```

## Evaluation signals

- All three output files (abundance matrix, sample metadata, metabolite metadata) exist on disk after the step.
- Re-importing each tab-delimited file with read.delim reproduces the exact dimensions of the original in-memory objects — no rows or columns dropped or altered.
- str(dat) on the returned list confirms three components (data, samples, features) were parsed, with the expected platform-specific annotation columns present.
- Passing the extracted components to Metaboprep(data=..., features=..., samples=...) yields a valid object (e.g. 100 samples × 100 features for the example dataset).
- Sample/feature metadata columns match the platform schema (e.g. run_day, box_id, lot for samples; platform, pathway, kegg, group_hmdb for features).

## Limitations

- The importers are platform-specific: read_metabolon targets Metabolon Excel files and read_nightingale targets Nightingale Health Excel files; other vendor formats (Olink .txt, SomaLogic .adat) are handled by different importers and require separate file-type parameters.
- The Metabolon importer requires the correct sheet name to be supplied explicitly (sheet='OrigScale'); reading the wrong sheet will not yield the intended abundance matrix.
- With return_Metaboprep=FALSE the function returns a plain list, not a Metaboprep object, so downstream QC functions cannot be called directly until the object is constructed with Metaboprep(data=, features=, samples=).

## Evidence

- [intro] Raw extraction writes tab-delimited datasets: "Read in and processes (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
- [readme] Importer returns list vs Metaboprep object: [evidence span withheld — anchor status: external_doc]
- [readme] Metabolon Excel sheet selection: "sheet             = "OrigScale", ## The name of the sheet in the excel file to read in"
- [abstract] Package extracts data from preformed worksheets: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
