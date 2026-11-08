---
name: csv-table-io
description: Use when a metabolomics processing step (normalization, imputation, scaling)
  needs a flat .csv or .xlsx table loaded into a DataFrame, or when a completed data
  frame must be persisted back to CSV for downstream steps or API consumers.
when_to_use_negative:
- Data is already loaded as an in-memory DataFrame and no file persistence or R bridge
  is needed
- The source data is a platform-specific raw format (e.g., instrument vendor output)
  rather than the post-annotation CSV/XLSX table Viime is designed to pick up
- The operation is web upload/drag-and-drop file ingestion into Viime's File Depot
  + SQLite (SQLAlchemy/File Depot), not programmatic CSV I/O
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Pandas
  role: Parses raw .csv/.xlsx metabolomics tables into DataFrames and writes processed
    tables back out as CSV
- name: OpenCPU
  role: Exposes R processing functions that take a CSV file path as argument and return
    a data frame; viime/opencpu.py handles pandas↔R frame communication
  repo: https://github.com/girder/viime
- name: Viime
  role: Application context whose backend leverages Pandas for raw file parsing and
    whose R package functions are defined on CSV files
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:d287bda87c0c4e6e1b73bd56c1a405fea339094c3af7cd210ac8c23632d60ab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - R
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# csv-table-io

## Summary

Reading and writing tabular metabolomics data as .csv (and .xlsx) files using Pandas DataFrames, as done in Viime's backend where Pandas handles raw file parsing and R/OpenCPU processing functions consume and return CSV paths and data frames.

## When to use

Use when a metabolomics processing step (normalization, imputation, scaling) needs a flat .csv or .xlsx table loaded into a DataFrame, or when a completed data frame must be persisted back to CSV for downstream steps or API consumers.

## When NOT to use

- Data is already loaded as an in-memory DataFrame and no file persistence or R bridge is needed
- The source data is a platform-specific raw format (e.g., instrument vendor output) rather than the post-annotation CSV/XLSX table Viime is designed to pick up
- The operation is web upload/drag-and-drop file ingestion into Viime's File Depot + SQLite (SQLAlchemy/File Depot), not programmatic CSV I/O

## Inputs

- Metabolomics concentration/data table as a .csv file (optionally .xlsx per Viime's upload interface)
- Column-role designation (primary ID, group, factor, masked/hidden, metabolite concentration columns)
- CSV file path passed to R/OpenCPU processing functions

## Outputs

- Pandas DataFrame holding the parsed table
- Processed data table written back out as a CSV file (e.g., scaled or fully imputed table)
- R data frame returned from OpenCPU-exposed functions and bridged back to pandas

## How to apply

Load the metabolomics concentration table with pandas.read_csv (or read_excel for .xlsx), designating which columns are the primary ID, group, factor, or metabolite concentration columns. Pass a CSV file path to R-based processing functions (e.g., missForest/impute imputation exposed via OpenCPU); per the Viime README, these functions 'should accept a path to a CSV file as an argument and return a data frame', with viime/opencpu.py handling the pandas↔R data frame conversion. After each processing step (imputation, scaling), write the result back with to_csv. Validate that the output CSV preserves the input's rows and columns and that post-imputation output contains no remaining missing values.

## Related tools

- **Pandas** (Parses raw .csv/.xlsx metabolomics tables into DataFrames and writes processed tables back out as CSV)
- **OpenCPU** (Exposes R processing functions that take a CSV file path as argument and return a data frame; viime/opencpu.py handles pandas↔R frame communication) — https://github.com/girder/viime
- **Viime** (Application context whose backend leverages Pandas for raw file parsing and whose R package functions are defined on CSV files) — https://github.com/girder/viime

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv'); df_scaled = df.apply(scale, axis=0); df_scaled.to_csv('scaled_metabolites.csv', index=False)
```

## Evaluation signals

- Output CSV exists and has the same rows and columns as the input table
- After imputation, the output CSV contains no remaining missing values
- A function exposed to the API server accepts a CSV path and returns a data frame (per the viime R package contract)
- Column roles (primary ID, group, factor, metabolite concentration) are preserved through the read/write round trip
- pandas↔R bridging via viime/opencpu.py round-trips the frame without schema loss

## Limitations

- R processing functions must be added to the custom viime R package and the docker image rebuilt to expose new methods to the API server
- Changes to the OpenCPU docker container's window position/size can terminate the service with SIGWINCH; run without a pseudo-terminal and keep stdin open
- Upload format support is limited to .xlsx and .csv in Viime's interface

## Evidence

- [abstract] The backend leverages Pandas for raw file parsing and normalization.: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] R functions take CSV paths and return data frames: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] pandas-R bridge: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
- [abstract] accepted upload formats: "we have designed an interactive drag and drop data upload interface which currently accepts .xlsx and .csv files."
- [abstract] OpenCPU R integration: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
