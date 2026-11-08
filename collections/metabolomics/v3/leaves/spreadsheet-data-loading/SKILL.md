---
name: spreadsheet-data-loading
description: Use when the starting material is a spreadsheet or delimited text file
  (e.g. .xlsx from the Instant Clue tutorial bundle) that must be ingested as a DataFrame
  before filtering, visualization, or row-wise fitting can begin.
when_to_use_negative:
- The data are not tabular spreadsheet/delimited text — e.g. optical recordings such
  as the Example Data 03 POMC neuron activity traces, which need their own handling
- The data are already loaded as a DataFrame and the goal is modeling (linear/polynomial/Michaelis-Menten
  fitting, correlation) rather than ingestion
- The data need batch-effect correction rather than loading — that is the role of
  the pyComBat/combat package, which must be installed separately
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Instant Clue
  role: Desktop GUI (PyQt6) whose upload path ingests .xlsx, .txt, .csv, .xml, .gz
    and .zip data for drag & drop visualization and analysis
  repo: https://github.com/hnolCol/instantclue
- name: Python
  role: Runtime (>=3.4 required by Instant Clue source) in which the spreadsheet is
    read into a pandas DataFrame for scripted workflows
  repo: https://github.com/hnolCol/instantclue
- name: pandas
  role: DataFrame library used to hold the loaded spreadsheet; version compatibility
    with combat must be respected when installing requirements.txt
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:a435148a4f6585e97a4efa9221962e7601215a361afc611474a7b567f6725963
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spreadsheet-data-loading

## Summary

Load tabular experimental data (e.g. the bundled Tutorial_Data_01.xlsx spreadsheet) into an analysis-ready form, either through Instant Clue's upload path (Excel, txt, csv, xml, gz, zip) or by reading the .xlsx into a pandas DataFrame for scripted activities such as curve fitting.

## When to use

Use when the starting material is a spreadsheet or delimited text file (e.g. .xlsx from the Instant Clue tutorial bundle) that must be ingested as a DataFrame before filtering, visualization, or row-wise fitting can begin.

## When NOT to use

- The data are not tabular spreadsheet/delimited text — e.g. optical recordings such as the Example Data 03 POMC neuron activity traces, which need their own handling
- The data are already loaded as a DataFrame and the goal is modeling (linear/polynomial/Michaelis-Menten fitting, correlation) rather than ingestion
- The data need batch-effect correction rather than loading — that is the role of the pyComBat/combat package, which must be installed separately

## Inputs

- Excel spreadsheet (.xlsx), e.g. Tutorial_Data_01.xlsx bundled with Instant Clue
- Tab-delimited text (.txt) or comma-separated (.csv) files
- Extensible Markup Language (.xml) files
- Compressed files (.gz, .zip)

## Outputs

- pandas DataFrame with one row per entity (protein/gene/condition) and named numeric columns
- Cleaned subset with missing-value rows removed prior to activities that cannot handle NaNs, leaving the source data unchanged

## How to apply

Identify the input format first: Instant Clue accepts Excel, tab-delimited .txt, .csv, .xml and compressed .gz/.zip files, so choose the matching reader (pd.read_excel for .xlsx). Load the file — for the tutorial workflow, Tutorial_Data_01.xlsx — into a pandas DataFrame, keeping column headers as the feature/sample names that later drag & drop activities or scripted fits will reference. Because Instant Clue's activities cannot handle missing values, pre-filter rows with NaN in the target numeric columns before submitting them to any activity; note that the software does this automatically 'without changing the source data', so scripted equivalents should preserve the raw DataFrame. When running Instant Clue from source rather than the Windows/Mac executables in the Releases section, create a virtual environment (python3 -m venv env), install combat first (it has very specific pandas/numpy versioning requirements), then pip install -r requirements.txt and start with python3 main.py.

## Related tools

- **Instant Clue** (Desktop GUI (PyQt6) whose upload path ingests .xlsx, .txt, .csv, .xml, .gz and .zip data for drag & drop visualization and analysis) — https://github.com/hnolCol/instantclue
- **Python** (Runtime (>=3.4 required by Instant Clue source) in which the spreadsheet is read into a pandas DataFrame for scripted workflows) — https://github.com/hnolCol/instantclue
- **pandas** (DataFrame library used to hold the loaded spreadsheet; version compatibility with combat must be respected when installing requirements.txt) — https://github.com/hnolCol/instantclue

## Examples

```
python3 -m venv env && source env/bin/activate && pip install -r requirements.txt && python3 main.py
```

## Evaluation signals

- The spreadsheet parses without error and the DataFrame's columns match the headers of Tutorial_Data_01.xlsx (e.g. 'Body weight', 'Condition')
- Rows with missing values in the target columns are removed before any activity that cannot handle missing values
- The original source file/DataFrame remains unchanged after pre-filtering, mirroring Instant Clue's 'without changing the source data' behaviour
- When launching from source, the virtual environment was built with combat installed first, then requirements.txt, and python3 main.py starts the application

## Limitations

- Activities in Instant Clue cannot handle missing values, so data are automatically pre-filtered — silent row loss is possible if NaNs are not inspected first
- The combat package is not in requirements.txt because of specific pandas/numpy versioning requirements; a numpy/pandas version-mismatch warning is expected during install
- Creating your own executable with PyInstaller 'requires some optimization'; the README recommends contacting the authors for customized hooks

## Evidence

- [results] Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip): "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files"
- [results] the accompanied "Tutorial_Data_01.xlsx": [evidence span withheld — anchor status: not_found]
- [results] if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity"
- [readme] First download the code and extract it. Open terminal (mac) or command line tool (windows) and navigate to instantclue/src/main/python: "First download the code and extract it. Open terminal (mac) or command line tool (windows) and navigate to instantclue/src/main/python"
- [readme] the package combat is not in the requirements table because it has very specific versioning requirements of pandas and numpy: "the package combat is not in the requirements table because it has very specific versioning requirements of pandas and numpy"
