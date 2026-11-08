---
name: csv-data-frame-io
description: Use when a metabolomics workflow step must read a concentration table
  from an uploaded .csv (or .xlsx) file, apply an element-wise operation in Python
  or R, and write the result back as a CSV with identical rows and columns.
when_to_use_negative:
- Input is not a tabular concentration table (e.g. raw instrument output like NMR
  spectra or MS peak lists) — Viime explicitly picks up the workflow after preprocessing
  and annotation.
- The table has not yet passed error checking (e.g. non-numeric concentration data
  or missing primary ID) and needs structure correction rather than value transformation.
- The operation requires preserving non-tabular metadata (e.g. spectral acquisition
  parameters) that a flat CSV cannot represent.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3077
tools:
- name: Pandas
  role: Python-side parsing of raw CSV/xlsx files and holding the concentration table
    as a data frame between processing steps
- name: R
  role: Hosts the exposed processing functions that read a CSV path and return a data
    frame for specialized operations
- name: OpenCPU
  role: Secondary REST service bridging pandas and R data frames so R functions can
    consume and emit CSV-backed tables
  repo: https://www.opencpu.org/
- name: Flask
  role: RESTful API backend that receives uploaded CSVs, dispatches processing, and
    returns processed CSV files
- name: Viime
  role: The web application whose upload interface accepts .xlsx/.csv files and whose
    pipeline consumes the CSV round-trip at each data treatment step
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:039bd4bc8d607e9705ad071d8238cd584d13b049c3998ec39eeacdc19fcd4b56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flask
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# csv-data-frame-io

## Summary

Round-trip metabolomics concentration tables between CSV files and tabular data frames (Pandas or R) so that each Viime processing step can be applied and persisted while preserving the primary ID and metadata columns.

## When to use

Use when a metabolomics workflow step must read a concentration table from an uploaded .csv (or .xlsx) file, apply an element-wise operation in Python or R, and write the result back as a CSV with identical rows and columns.

## When NOT to use

- Input is not a tabular concentration table (e.g. raw instrument output like NMR spectra or MS peak lists) — Viime explicitly picks up the workflow after preprocessing and annotation.
- The table has not yet passed error checking (e.g. non-numeric concentration data or missing primary ID) and needs structure correction rather than value transformation.
- The operation requires preserving non-tabular metadata (e.g. spectral acquisition parameters) that a flat CSV cannot represent.

## Inputs

- uploaded .csv (or .xlsx) metabolomics concentration table with a primary ID column, optional factor/group/masked columns, and metabolite concentration columns
- file path to a CSV on disk for R-side processing functions

## Outputs

- CSV file containing the processed concentration table with the same rows and columns as the input (structure preserved, values transformed)
- in-memory Pandas or R data frame ready for the next Viime processing step

## How to apply

Load the post-normalization metabolomics concentration table with Pandas, treating one column as the primary ID and the remaining metabolite columns as concentration values; the backend leverages Pandas for raw file parsing and normalization. When the operation must run in R, the R function accepts a path to a CSV file as its argument and returns a data frame, with the OpenCPU bridge handling communication between pandas and R data frames. Apply the chosen transformation or scaling element-wise to the metabolite concentration columns only, keeping ID, factor, group, and masked/hidden columns untouched. Write the transformed data frame back to CSV via the API server, preserving the original row and column structure. Correctness hinges on the output CSV remaining a valid, structure-preserving table that downstream Viime steps (imputation, transformation, scaling) can consume.

## Related tools

- **Pandas** (Python-side parsing of raw CSV/xlsx files and holding the concentration table as a data frame between processing steps)
- **R** (Hosts the exposed processing functions that read a CSV path and return a data frame for specialized operations)
- **OpenCPU** (Secondary REST service bridging pandas and R data frames so R functions can consume and emit CSV-backed tables) — https://www.opencpu.org/
- **Flask** (RESTful API backend that receives uploaded CSVs, dispatches processing, and returns processed CSV files)
- **Viime** (The web application whose upload interface accepts .xlsx/.csv files and whose pipeline consumes the CSV round-trip at each data treatment step) — https://github.com/girder/viime

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('normalized.csv'); df[num_cols] = df[num_cols].apply(np.log10); df.to_csv('transformed.csv', index=False)"
```

## Evaluation signals

- Output CSV parses cleanly with Pandas and has the same number of rows and columns as the input table.
- The primary ID column is unchanged and unique after the round-trip.
- Transformed concentration values match the chosen operation (e.g. Log10, Log2, sqrt, cube root) applied element-wise to the input values.
- Non-concentration columns (factor, group, masked/hidden) are byte-identical to the input.
- The file can be uploaded back into Viime via its drag-and-drop .csv interface without structure errors.

## Limitations

- R-side functions must accept a CSV path and return a data frame; adding new exposed methods requires modifying the viime R package and rebuilding the Docker image.
- The pandas↔R bridge is described as work-in-progress (viime/opencpu.py), so complex dtypes or non-tabular structures may not survive the round-trip.
- Viime's intent is to pick up the workflow after preprocessing and annotation — raw unprocessed instrument files are out of scope for this skill.

## Evidence

- [abstract] backend leverages Pandas for raw file parsing and normalization: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] Functions contained in this package should accept a path to a CSV file as an argument and return a data frame.: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames.: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
- [readme] you can use it to upload the CSV files found in the [sample data](samples/data) directory: "Once the client is running, you can use it to upload the CSV files found
in the [sample data](samples/data) directory of this repository."
- [abstract] The intent of Viime package is to pick up the workflow after preprocessing and annotation: "The intent of Viime package is to pick up the workflow after preprocessing and annotation"
- [abstract] interactive drag and drop data upload interface which currently accepts .xlsx and .csv files: "we have designed an interactive drag and drop data upload interface which currently accepts .xlsx and .csv files."
