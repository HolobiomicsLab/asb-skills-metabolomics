---
name: csv-xlsx-tabular-parsing
description: Use when an uploaded metabolomics dataset arrives as a .csv or .xlsx
  table and must be parsed and checked for missing data, non-numeric concentration
  values, and primary-ID problems before processing.
when_to_use_negative:
- Input is already a cleaned, imputed, normalized table ready for transformation or
  scaling — Viime's validation targets raw uploads where errors still need correction
- Data originate from raw instrument output (e.g. MS or NMR spectra) that still needs
  preprocessing and annotation — Viime picks up the workflow after those stages
- Tabular data are not metabolomics concentration measurements (e.g. generic non-spectrometry
  spreadsheets without metabolite concentration columns)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3077
tools:
- name: Viime
  role: Web application whose upload-time validation detects missing-data, non-numeric
    concentration, and primary-ID errors in parsed tables
  repo: https://github.com/girder/viime
- name: Pandas
  role: Backend library used for raw file parsing of the .csv/.xlsx uploads
- name: Flask
  role: RESTful API framework hosting the processing backend that serves the parsed/validated
    data
- name: SQLAlchemy
  role: ORM persisting uploaded files and associated data in the SQLite database
- name: File Depot
  role: Links stored files with database rows via custom fields
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:3430ab8b9878d111e9d99f8841ba469abfd975147ce25559db30fc8b017e6312
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - File Depot
  - Flask
  - Pandas
  - SQLAlchemy
  - Viime
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# csv-xlsx-tabular-parsing

## Summary

Parse raw metabolomics tables (.csv or .xlsx) with Pandas inside Viime's Flask backend, then run validation that flags missing-data levels beyond a default threshold, non-numeric concentration values, missing or non-unique primary IDs, and low-variance metabolites.

## When to use

Use when an uploaded metabolomics dataset arrives as a .csv or .xlsx table and must be parsed and checked for missing data, non-numeric concentration values, and primary-ID problems before processing.

## When NOT to use

- Input is already a cleaned, imputed, normalized table ready for transformation or scaling — Viime's validation targets raw uploads where errors still need correction
- Data originate from raw instrument output (e.g. MS or NMR spectra) that still needs preprocessing and annotation — Viime picks up the workflow after those stages
- Tabular data are not metabolomics concentration measurements (e.g. generic non-spectrometry spreadsheets without metabolite concentration columns)

## Inputs

- Uploaded .csv metabolomics table
- Uploaded .xlsx metabolomics table

## Outputs

- Structured validation report of errors (missing-data threshold, non-numeric concentration data, missing/non-unique primary ID) keyed by row/column
- Warnings for too many missing/'not a number' values and low-variance metabolite columns
- Parsed Pandas table with designated column roles (primary ID, masked/hidden, factor, group, metabolite concentration)

## How to apply

Load the uploaded .csv or .xlsx file with Pandas in Viime's Flask backend, identifying the primary ID column, factor/group columns, and metabolite concentration columns. Validate the parsed table: flag levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration columns, the lack of a primary ID, and a non-unique primary ID. Warn separately on likely problems such as too many missing or 'not a number' values within a metabolite or group, and compute per-column variance to warn on low-variance metabolites (e.g. one flagged for possible omission, per Figure 2). Present errors so the user can correct them and designate column roles (primary ID, masked/hidden, factor, group, metabolite concentration); once errors are corrected, automatic imputation proceeds. Store the uploaded file and associated metadata in SQLite via SQLAlchemy's ORM, linking files to rows with File Depot custom fields.

## Related tools

- **Viime** (Web application whose upload-time validation detects missing-data, non-numeric concentration, and primary-ID errors in parsed tables) — https://github.com/girder/viime
- **Pandas** (Backend library used for raw file parsing of the .csv/.xlsx uploads)
- **Flask** (RESTful API framework hosting the processing backend that serves the parsed/validated data)
- **SQLAlchemy** (ORM persisting uploaded files and associated data in the SQLite database)
- **File Depot** (Links stored files with database rows via custom fields)

## Examples

```
flask run  # then open the web client and drag-and-drop a samples/data CSV to trigger upload-time validation
```

## Evaluation signals

- Every uploaded .csv/.xlsx table parses without unhandled exceptions and yields a designated primary ID, group/factor, and metabolite concentration columns
- A table with missing data above the default threshold within a group or across all samples produces an error entry
- Non-numeric values injected into a concentration column are flagged as errors
- Duplicated or absent primary IDs produce explicit error reports
- Low-variance metabolite columns and metabolite/group groups with excessive missing or 'not a number' values produce warnings rather than silent passes

## Limitations

- Validation applies a default missing-data threshold that may need user adjustment; the user corrects errors interactively after upload
- Annotation is not required — Viime picks up the workflow after preprocessing and annotation, so raw, unannotated instrument output is out of scope

## Evidence

- [abstract] Upload validation checks: "Errors that are detected include levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration data, the lack of a primary ID"
- [abstract] Warning classes: [evidence span withheld — anchor status: not_found]
- [abstract] Accepted formats: "an interactive drag and drop data upload interface which currently accepts .xlsx and .csv files"
- [abstract] Column role designation: "designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [abstract] Backend parsing stack: "The backend leverages Pandas for raw file parsing and normalization."
- [abstract] Persistence layer: [evidence span withheld — anchor status: not_found]
- [abstract] File linking: "Files stored internally are linked with rows in the database using custom fields provided by File Depot"
- [abstract] Downstream step after correction: "Once errors are corrected, data imputation is automatically performed."
- [abstract] Workflow scope: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
- [readme] README upload demo: "Once the client is running, you can use it to upload the CSV files found in the [sample data](samples/data) directory of this repository."
