---
name: metabolite-abundance-extraction-from-vendor-worksheets
description: Use when source metabolomics data arrive as vendor worksheets (e.g. Metabolon
  Excel 'OrigScale' sheet or Nightingale Health .xlsx) and you need the abundance
  matrix, sample metadata, and metabolite metadata as separate tab-delimited datasets.
when_to_use_negative:
- Input is already a flat feature table or generic CSV (e.g. the packaged dummy_data.csv
  / dummy_samples.csv / dummy_features.csv) — no vendor worksheet parsing is needed,
  so assemble the Metaboprep object directly from the three tables.
- The goal is quality filtering or outlier removal (sample/feature missingness, total_peak_area_sd,
  PCA outliers) — that is handled by metaboprep's quality_control pipeline, which
  operates on data after extraction.
- A Metaboprep object is needed immediately — set return_Metaboprep=TRUE or build
  it with Metaboprep(data=..., features=..., samples=...) rather than exporting raw
  tab-delimited files.
edam_topics: []
tools:
- name: metaboprep
  role: Provides the platform-specific importers (read_metabolon, read_nightingale)
    that split vendor worksheets into data, samples, and features components for tab-delimited
    export
  repo: github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep importers and write.table/read.delim
    export-validation are executed
  repo: https://github.com/MRCIEU/metaboprep
- name: readxl
  role: Underlying Excel reader invoked by metaboprep's read_metabolon importer when
    parsing the 'OrigScale' sheet of Metabolon .xlsx files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:0ece6b3abc1ecf0c33a0c028bd0e88e13378ee628aceed53c093813d4123d2ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - metaboprep
  - readxl
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-abundance-extraction-from-vendor-worksheets

## Summary

Extracts raw metabolite abundance matrices plus sample and metabolite (feature) metadata from vendor-format source files (e.g. Nightingale Health or Metabolon Excel workbooks) using metaboprep's platform-specific importers, then writes the three components as unaltered tab-delimited text files for downstream use.

## When to use

Use when source metabolomics data arrive as vendor worksheets (e.g. Metabolon Excel 'OrigScale' sheet or Nightingale Health .xlsx) and you need the abundance matrix, sample metadata, and metabolite metadata as separate tab-delimited datasets.

## When NOT to use

- Input is already a flat feature table or generic CSV (e.g. the packaged dummy_data.csv / dummy_samples.csv / dummy_features.csv) — no vendor worksheet parsing is needed, so assemble the Metaboprep object directly from the three tables.
- The goal is quality filtering or outlier removal (sample/feature missingness, total_peak_area_sd, PCA outliers) — that is handled by metaboprep's quality_control pipeline, which operates on data after extraction.
- A Metaboprep object is needed immediately — set return_Metaboprep=TRUE or build it with Metaboprep(data=..., features=..., samples=...) rather than exporting raw tab-delimited files.

## Inputs

- Nightingale Health Excel workbook (.xlsx), e.g. nightingale_v1_example.xlsx
- Metabolon Excel workbook (.xlsx) with results on the 'OrigScale' sheet, e.g. metabolon_v1.1_example.xlsx
- Other metaboprep-supported vendor formats: olink_v1_example.txt, somalogic_v1_example.adat

## Outputs

- Metabolite abundance matrix as a tab-delimited text file
- Sample metadata table (e.g. technical batch/run_day, box_id, lot, sample identifiers) as a tab-delimited text file
- Metabolite/feature metadata table (e.g. metabolite_id, platform, pathway, kegg) as a tab-delimited text file

## How to apply

Load metaboprep in R and locate the packaged example input (e.g. metabolon_v1.1_example.xlsx or nightingale_v1_example.xlsx) via system.file('extdata', ...). Call the platform-specific importer — read_metabolon for Metabolon Excel with sheet='OrigScale', or read_nightingale for Nightingale Health Excel — with return_Metaboprep=FALSE so the raw data, samples, and features components come back as separate list elements rather than a Metaboprep object. Inspect the returned structure (str(dat)) to confirm the abundance matrix, sample table, and feature table parsed correctly, noting platform-specific annotation columns such as xenobiotic super_pathway membership and derived-measure flags. Write each component unaltered to disk with write.table(sep='\t', quote=FALSE), then re-import with read.delim and confirm row/column counts match the in-memory objects. This corresponds to metaboprep's stated goal of reading in metabolite data and saving datasets in tab-delimited format for use elsewhere; downstream quality filtering (quality_control with thresholds like sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5) is a separate step applied after extraction.

## Related tools

- **metaboprep** (Provides the platform-specific importers (read_metabolon, read_nightingale) that split vendor worksheets into data, samples, and features components for tab-delimited export) — github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep importers and write.table/read.delim export-validation are executed) — https://github.com/MRCIEU/metaboprep
- **readxl** (Underlying Excel reader invoked by metaboprep's read_metabolon importer when parsing the 'OrigScale' sheet of Metabolon .xlsx files)

## Examples

```
mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"), sheet = "OrigScale", return_Metaboprep = FALSE)
```

## Evaluation signals

- All three output files (abundance matrix, sample metadata, feature metadata) exist on disk after the run.
- Re-importing each file with read.delim reproduces the exact dimensions of the in-memory abundance matrix, sample table, and feature table — no rows or columns dropped or altered.
- str(dat) on the importer output shows the expected three components (data, samples, features) with platform-specific annotation columns preserved (e.g. xenobiotic super_pathway membership, derived-measure flags).
- str(read.delim(...)) on the written files confirms tab-delimited structure and unaltered values.

## Limitations

- The package lifecycle is marked 'experimental', so the extraction API may change between versions.
- Importers are platform-specific and format-sensitive: Metabolon data must be read from the 'OrigScale' sheet name, and unsupported layouts will not parse into the data/samples/features triple.
- Extraction only handles the read-in step; it does not itself perform quality selection or filtering — those require the separate quality_control pipeline with user-defined thresholds.
- When running the downstream PCA outlier analysis, metaboprep warns if the stated max_num_pcs exceeds the number of available informative PCs (e.g. max_num_pcs=10 vs 2 informative PCs).

## Evidence

- [intro] Read in and process (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere: "Read in and process (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
- [abstract] The package extracts data from preformed worksheets: "The package extracts data from preformed worksheets"
- [methods] read_metabolon system.file extdata metabolon_v1.1_example.xlsx: "read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx""
- [readme] README example: import data with read_metabolon sheet OrigScale return_Metaboprep FALSE: "sheet             = "OrigScale", ## The name of the sheet in the excel file to read in"
- [readme] README lifecycle badge: "[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)"
