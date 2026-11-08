---
name: spreadsheet-table-import
description: Use when the raw input is a spreadsheet or delimited table (e.g. 'Tutorial_Data_01.xlsx')
  that must be loaded with pandas/Python >=3.4 or Instant Clue so its measurement
  columns and categorical annotation columns can be joined and filtered.
when_to_use_negative:
- The data are already an analysis-ready feature table loaded in memory and only need
  column-level activities such as normalization, imputation or smoothing
- The data are not tabular spreadsheet content (e.g. optical recordings from Example
  Data 03-style time series stored in non-table formats)
- The goal is batch-effect correction rather than import — that requires the pyComBat
  package, which is not part of the standard requirements table
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI used to upload spreadsheet tables of various file types for visual
    and statistical analysis
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Scripted pandas-based loading of the Excel table and export of derived filtered/annotated
    CSVs
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:459e272858bd1f9fb58e6202a8555b38bc1072bf23a79a68a92ddbf70003e54d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spreadsheet-table-import

## Summary

Import tabular experimental data from spreadsheet-style files (Excel, csv, txt, xml, gz, zip) into a pandas/Python or Instant Clue analysis session, selecting the columns that carry measurements and categorical annotation terms (e.g. GO terms) for downstream filtering and annotation.

## When to use

Use when the raw input is a spreadsheet or delimited table (e.g. 'Tutorial_Data_01.xlsx') that must be loaded with pandas/Python >=3.4 or Instant Clue so its measurement columns and categorical annotation columns can be joined and filtered.

## When NOT to use

- The data are already an analysis-ready feature table loaded in memory and only need column-level activities such as normalization, imputation or smoothing
- The data are not tabular spreadsheet content (e.g. optical recordings from Example Data 03-style time series stored in non-table formats)
- The goal is batch-effect correction rather than import — that requires the pyComBat package, which is not part of the standard requirements table

## Inputs

- Excel workbook (e.g. Tutorial_Data_01.xlsx) with columns carrying measurement values and categorical annotation terms
- Tab-delimited .txt or .csv table of omics or quantitative data
- XML or compressed (.gz/.zip) tabular data file

## Outputs

- In-memory DataFrame / Instant Clue dataset with measurement and annotation columns typed and ready for filtering
- Derived CSV exports (e.g. filtered_annotated_table.csv) that leave the source table unchanged

## How to apply

Load the source table with pandas or Instant Clue — Instant Clue accepts 'Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)'. Read the columns that carry the measurement values and the columns carrying categorical annotation terms (e.g. GO terms) so every row carries its category term(s), mirroring the join step of Instant Clue's 'Find Category & Annotate' filter. Keep the source table unmodified: the article's convention is that filtering and annotation 'without changing the source data', so write derived results (e.g. filtered_annotated_table.csv) to new files instead of overwriting the input. For scripted use with the Instant Clue source, create a virtual environment and install requirements.txt first, since pyComBat has 'very specific versioning requirements of pandas and numpy'.

## Related tools

- **Instant Clue** (Desktop GUI used to upload spreadsheet tables of various file types for visual and statistical analysis) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Scripted pandas-based loading of the Excel table and export of derived filtered/annotated CSVs) — https://github.com/hnolCol/instantclue

## Examples

```
import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx')
```

## Evaluation signals

- The loaded table parses without error and the expected annotation/category field is present (field_present check)
- Row count and column set of the in-memory table match the source file — the source data must remain unchanged
- Output files such as filtered_annotated_table.csv exist and parse as CSV (file_exists, file_format_is checks)
- Column dtypes are correct: numeric measurement columns parsed as numbers, categorical annotation columns (e.g. GO terms) as strings
- Every retained/derived row matches the expected categorical condition applied after import

## Limitations

- pyComBat is 'not in the requirements table because it has very specific versioning requirements of pandas and numpy', so installing all dependencies naively can cause numpy/pandas version mismatch warnings
- Building your own executable with Pyinstaller 'requires some optimization'; the maintainers recommend contacting them for customized hooks
- Only file-based tabular inputs are supported for upload — no direct database connectors are described

## Evidence

- [results] Upload file types: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [intro] Python version requirement: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
- [results] Tutorial dataset: [evidence span withheld — anchor status: not_found]
- [results] Source data preservation: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [readme] Stack and GUI library: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
