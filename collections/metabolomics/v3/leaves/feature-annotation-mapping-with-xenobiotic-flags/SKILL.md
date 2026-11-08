---
name: feature-annotation-mapping-with-xenobiotic-flags
description: Use when a raw metabolomics workbook (nightingale_v1_example.xlsx, metabolon_v1.1_example.xlsx)
  must be decomposed into abundance, sample, and feature metadata tables with annotation
  columns like super_pathway retained unaltered.
when_to_use_negative:
- The input is already a tab-delimited feature table with annotations present — no
  importer extraction is needed
- The goal is quality-control filtering (sample/feature missingness, PCA outliers)
  rather than raw data extraction and annotation mapping
- The source file is not a supported platform format (e.g. no preformed worksheets
  for the importer to parse)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Provides the platform-specific importers (read_nightingale, read_metabolon)
    that decompose preformed worksheets into data, samples, and features components
  repo: github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep importers and write.table/read.delim
    validation are executed
- name: readxl
  role: Underlying Excel reader used by read_metabolon to parse Metabolon workbook
    sheets such as 'OrigScale'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:8690691bcb7ea8b839827e08a3c2e226adfc7d7d9c004913949882d518b7ee6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - metaboprep
  - readxl
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Feature annotation mapping with xenobiotic flags

## Summary

Extracts the metabolite (feature) metadata table from platform-specific metabolomics source files (e.g. Nightingale Health or Metabolon Excel workbooks) via metaboprep importers, preserving platform-specific annotation columns such as xenobiotic super_pathway membership and derived-measure flags, and exports them as tab-delimited text for downstream use.

## When to use

Use when a raw metabolomics workbook (nightingale_v1_example.xlsx, metabolon_v1.1_example.xlsx) must be decomposed into abundance, sample, and feature metadata tables with annotation columns like super_pathway retained unaltered.

## When NOT to use

- The input is already a tab-delimited feature table with annotations present — no importer extraction is needed
- The goal is quality-control filtering (sample/feature missingness, PCA outliers) rather than raw data extraction and annotation mapping
- The source file is not a supported platform format (e.g. no preformed worksheets for the importer to parse)

## Inputs

- Nightingale Health Excel workbook (e.g. nightingale_v1_example.xlsx)
- Metabolon Excel workbook with sheet='OrigScale' (e.g. metabolon_v1.1_example.xlsx)
- metaboprep package data importers (read_nightingale, read_metabolon)

## Outputs

- Tab-delimited metabolite (feature) metadata file including annotation columns (super_pathway, derived-measure flags)
- Optionally, tab-delimited abundance matrix and sample metadata files
- Validation check confirming re-imported files reproduce original dimensions

## How to apply

Load metaboprep in R and locate the packaged example workbook via system.file('extdata', ...). Call the platform-specific importer — read_nightingale for Nightingale Health Excel files or read_metabolon (sheet='OrigScale') for Metabolon files — with return_Metaboprep=FALSE so the data, samples, and features components are returned as separate objects rather than a Metaboprep object. Inspect the structure with str(dat) to confirm the abundance matrix, sample table, and feature table parsed correctly, and record platform-specific annotation columns such as xenobiotic super_pathway membership and derived-measure flags. Write the unaltered feature metadata (and optionally the abundance matrix and sample metadata) to disk with write.table(sep='\t', quote=FALSE), then validate by re-importing with read.delim and confirming row/column counts match the in-memory objects with nothing dropped or altered.

## Related tools

- **metaboprep** (Provides the platform-specific importers (read_nightingale, read_metabolon) that decompose preformed worksheets into data, samples, and features components) — github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep importers and write.table/read.delim validation are executed)
- **readxl** (Underlying Excel reader used by read_metabolon to parse Metabolon workbook sheets such as 'OrigScale')

## Examples

```
dat <- read_nightingale(system.file("extdata", "nightingale_v1_example.xlsx", package = "metaboprep"), return_Metaboprep = FALSE)
```

## Evaluation signals

- The three output files exist and, when re-imported with read.delim, reproduce the dimensions of the original abundance matrix, sample table, and feature table with no rows or columns dropped or altered
- str(dat) shows a correctly parsed abundance matrix, sample table, and feature table before export
- Platform-specific annotation columns (xenobiotic super_pathway membership, derived-measure flags) are present in the exported feature metadata
- Files written with write.table(sep='\t', quote=FALSE) round-trip without mangled fields or changed quoting

## Limitations

- Importer coverage is platform-specific: read_nightingale and read_metabolon expect preformed worksheets in their respective Excel layouts; other formats (olink_v1_example.txt, somalogic_v1_example.adat, dummy_data.csv) need separate handling
- The package lifecycle is experimental, and extraction targets unaltered raw data — filtering and threshold-based exclusion happen in a later quality_control step, not here
- If the Excel workbook lacks the expected sheet (e.g. 'OrigScale' for Metabolon), the importer cannot parse the components

## Evidence

- [abstract] extraction from preformed worksheets: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
- [intro] tab-delimited export goal: "Read in and processes (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
- [readme] importer invocation pattern: "mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"), sheet = "OrigScale", return_Metaboprep = FALSE"
