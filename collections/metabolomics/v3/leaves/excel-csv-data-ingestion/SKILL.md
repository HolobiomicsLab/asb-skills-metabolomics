---
name: excel-csv-data-ingestion
description: Use when a metabolomics feature table arrives as csv or Excel (e.g.,
  a cancerCell demo table or an XCMS/MS-DIAL/MZmine/Compound Discoverer/Progenesis
  QI export) and must be parsed with metadata before preprocessing.
when_to_use_negative:
- Input is raw instrument spectra or vendor files (e.g., mzML/Raw) rather than a processed
  feature table — MetaboReport's uploading module consumes csv/Excel feature tables,
  not raw MS data
- The table has already been normalized and metadata attached (parsed_feature_table.csv
  already exists and validates) — re-ingestion risks duplicating samples or features
- The analysis goal is downstream statistics, visualization, or HTML report generation
  — this skill only covers the ingestion/normalization stage preceding MetaboReport
  preprocessing
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboReport
  role: Shiny application whose data uploading module defines the supported csv/Excel
    feature-table formats and header conventions being reproduced
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime for reading the csv/Excel feature table, normalizing headers, and
    writing the parsed csv output
- name: readxl
  role: R package used to read the Excel-format feature table input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:75b2c296a8ac04a05da9376ad9cbf8c00051240a259317ebb48131e6c30b2c2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - readxl
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# excel-csv-data-ingestion

## Summary

Ingest metabolomics feature tables (features in rows, samples in columns) from csv or Excel files, including exports from XCMS, MS-DIAL, MZmine, Compound Discoverer, and Progenesis QI, normalizing tool-specific header conventions and attaching sample metadata.

## When to use

Use when a metabolomics feature table arrives as csv or Excel (e.g., a cancerCell demo table or an XCMS/MS-DIAL/MZmine/Compound Discoverer/Progenesis QI export) and must be parsed with metadata before preprocessing.

## When NOT to use

- Input is raw instrument spectra or vendor files (e.g., mzML/Raw) rather than a processed feature table — MetaboReport's uploading module consumes csv/Excel feature tables, not raw MS data
- The table has already been normalized and metadata attached (parsed_feature_table.csv already exists and validates) — re-ingestion risks duplicating samples or features
- The analysis goal is downstream statistics, visualization, or HTML report generation — this skill only covers the ingestion/normalization stage preceding MetaboReport preprocessing

## Inputs

- Metabolomics feature table in csv format (features in rows, samples in columns)
- Feature table in Excel format (read via R readxl)
- Tool-specific exports from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI
- Separate sample metadata table (optional, when metadata is not embedded in sample column names)

## Outputs

- parsed_feature_table.csv containing the normalized feature table with sample columns and attached metadata fields

## How to apply

Load the feature table with R, using the readxl package when the input is an Excel file and csv readers for delimited text. Recognize that sample metadata may either be embedded in the sample column names (common in XCMS, MS-DIAL, MZmine, Compound Discoverer, and Progenesis QI exports) or supplied as a separate metadata table, and derive the design accordingly. Normalize tool-specific header conventions into a consistent parsed structure without fabricating sample or feature entries. Write the normalized parsed feature table to parsed_feature_table.csv using R's csv output functions, then verify the output parses cleanly and matches the input demo table's samples and metadata fields.

## Related tools

- **MetaboReport** (Shiny application whose data uploading module defines the supported csv/Excel feature-table formats and header conventions being reproduced) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime for reading the csv/Excel feature table, normalizing headers, and writing the parsed csv output)
- **readxl** (R package used to read the Excel-format feature table input)

## Examples

```
ft <- readxl::read_excel("cancerCell.xlsx"); write.csv(ft, "parsed_feature_table.csv", row.names = FALSE)
```

## Evaluation signals

- parsed_feature_table.csv exists and parses cleanly in R without errors
- Sample columns in the output csv exactly match the sample columns of the input demo feature table (no fabricated sample entries)
- Feature entries in the output match the input table's features (no fabricated feature entries)
- Attached metadata fields are consistent with the metadata embedded in the sample column names or the separate metadata table
- Tool-specific header conventions (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) have been normalized to a single consistent schema

## Limitations

- Only csv and Excel feature-table inputs are supported; raw spectra or other formats are out of scope
- Sample metadata must either be embedded in the sample column names or provided as a separate metadata table; other metadata layouts are not handled
- Header conventions differ across the five supported tools (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) and must be normalized per tool before merging

## Evidence

- [other] The data uploading module accepts feature tables in csv or Excel format, and sample metadata can either be embedded in the feature table's sample column names: [evidence span withheld — anchor status: not_found]
- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [methods] remotes::install_github("YonghuiDong/MetaboReport"): "remotes::install_github("YonghuiDong/MetaboReport")"
