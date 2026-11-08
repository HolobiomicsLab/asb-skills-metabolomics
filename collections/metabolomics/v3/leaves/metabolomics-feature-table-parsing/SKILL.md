---
name: metabolomics-feature-table-parsing
description: Use when raw preprocessing tool exports (XCMS, MS-DIAL, MZmine, Compound
  Discoverer, Progenesis QI) or csv/Excel feature tables must be parsed and normalized
  into a feature-table-plus-metadata structure before metabolomics statistical analysis
  or reporting.
when_to_use_negative:
- Input is already a normalized, tool-agnostic feature table with consistent sample
  columns and metadata — re-parsing adds no value
- The data are raw instrument files (e.g. mzML/NetCDF) that still need peak picking
  by a preprocessing tool like XCMS or MS-DIAL; this skill starts from the tool's
  feature-table output
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboReport
  role: Target application whose data uploading module defines the accepted feature-table
    format and metadata conventions (csv/Excel, embedded or separate metadata)
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Language used to load the feature table, normalize tool-specific headers,
    and write the parsed csv
- name: readxl
  role: R package used to read Excel-format feature tables during upload
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:90d6af262e523cc5e057346decf137b79162b13a00b0e31ccea93a4fe4c77cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - Azimuth DB
  - MetaboReport
  - R
  - metDataModel
  - readxl
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-feature-table-input-parsing
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-feature-table-parsing

## Summary

Loads and normalizes metabolomics feature tables exported by common processing tools (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) into a clean samples-in-columns/features-in-rows table with attached sample metadata, as required for MetaboReport upload.

## When to use

Use when raw preprocessing tool exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) or csv/Excel feature tables must be parsed and normalized into a feature-table-plus-metadata structure before metabolomics statistical analysis or reporting.

## When NOT to use

- Input is already a normalized, tool-agnostic feature table with consistent sample columns and metadata — re-parsing adds no value
- The data are raw instrument files (e.g. mzML/NetCDF) that still need peak picking by a preprocessing tool like XCMS or MS-DIAL; this skill starts from the tool's feature-table output

## Inputs

- Feature table in csv or Excel format (features in rows, samples in columns), e.g. the cancerCell demo metabolomics feature table
- Tool-specific exports from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI
- Optional separate sample metadata table (or metadata embedded in sample column names)

## Outputs

- parsed_feature_table.csv — normalized feature table with sample columns and attached metadata fields
- Sample metadata derived from column names or the supplied metadata table

## How to apply

Read the feature table (features in rows, samples in columns) from csv or Excel — use R's readxl package for Excel input. Account for the tool-specific header conventions of the source export (XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI) and derive sample metadata either from the sample column names or from a separate metadata table. Normalize the parsed table so sample columns and metadata fields are consistent, then write the result to parsed_feature_table.csv with R's csv output functions. Validate that the output csv parses cleanly and that its sample columns and attached metadata fields match the input table exactly, with no fabricated sample or feature entries.

## Related tools

- **MetaboReport** (Target application whose data uploading module defines the accepted feature-table format and metadata conventions (csv/Excel, embedded or separate metadata)) — https://github.com/YonghuiDong/MetaboReport
- **R** (Language used to load the feature table, normalize tool-specific headers, and write the parsed csv)
- **readxl** (R package used to read Excel-format feature tables during upload)

## Examples

```
library(readxl); ft <- read_excel("cancerCell.xlsx"); write.csv(ft, "parsed_feature_table.csv", row.names = FALSE)
```

## Evaluation signals

- parsed_feature_table.csv exists and parses cleanly as csv
- Sample columns of the output match the sample columns of the input demo table (no fabricated sample entries)
- Feature rows of the output match the input table (no fabricated feature entries)
- Attached metadata fields are consistent with metadata derived from input column names or the separate metadata table
- Tool-specific header conventions of the source export (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) are correctly normalized

## Limitations

- Sample metadata must either be embedded in the sample column names or supplied as a separate metadata table; other metadata provenance is not supported by the upload module
- Only csv and Excel feature-table formats are accepted; raw instrument files are out of scope for the data uploading module

## Evidence

- [other] The data uploading module accepts feature tables in csv or Excel format, and sample metadata can either be embedded in the feature table's sample column names or supplied via a separate metadata table.: [evidence span withheld — anchor status: not_found]
- [other] Load the cancerCell demo metabolomics feature table (features in rows, samples in columns) from csv or Excel using R with the readxl package for Excel input.: [evidence span withheld — anchor status: not_found]
- [other] Normalize the tool-specific header conventions of the supported exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI), deriving sample metadata either from column names or from a separate metadata table.: [evidence span withheld — anchor status: not_found]
- [intro] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [abstract] MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development: "MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development"
