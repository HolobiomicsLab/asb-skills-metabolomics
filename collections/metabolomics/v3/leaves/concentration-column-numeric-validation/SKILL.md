---
name: concentration-column-numeric-validation
description: Use when a metabolomics table (.csv or .xlsx) has been uploaded to Viime
  and designated concentration columns must be checked for non-numeric values before
  imputation, normalization, or analysis can proceed.
when_to_use_negative:
- Input is already validated, imputed, and normalized — validation runs only at the
  upload stage before error correction and imputation.
- The file's columns have not been designated as concentration/primary ID/group columns
  — without user designation there is nothing to validate against.
- The goal is statistical analysis (ANOVA, volcano plots, heatmaps) on an already-processed
  dataset rather than pre-flight error detection on raw uploaded tables.
edam_operation: http://edamontology.org/operation_3354
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Viime
  role: 'Hosts the upload-time validation: detects errors including non-numeric data
    in concentration columns during the drag-and-drop upload page''s error-correction
    workflow'
  repo: girder/viime
- name: Pandas
  role: Backend library that parses the raw uploaded .csv/.xlsx files before validation
    checks are applied
  repo: girder/viime
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:00e3d6c04383534872a6e1299adbf64d07eb1e09f24c918f986c6333845f8ee8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_3354
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# concentration-column-numeric-validation

## Summary

A validation routine that scans designated metabolite concentration columns in uploaded .csv or .xlsx metabolomics tables and flags non-numeric values as errors blocking downstream processing, alongside related checks (missing-data threshold, primary ID uniqueness, low-variance warnings).

## When to use

Use when a metabolomics table (.csv or .xlsx) has been uploaded to Viime and designated concentration columns must be checked for non-numeric values before imputation, normalization, or analysis can proceed.

## When NOT to use

- Input is already validated, imputed, and normalized — validation runs only at the upload stage before error correction and imputation.
- The file's columns have not been designated as concentration/primary ID/group columns — without user designation there is nothing to validate against.
- The goal is statistical analysis (ANOVA, volcano plots, heatmaps) on an already-processed dataset rather than pre-flight error detection on raw uploaded tables.

## Inputs

- uploaded .csv or .xlsx metabolomics table
- column-type designations (primary ID, factor, group, metabolite concentration) set by the user in the Viime upload interface

## Outputs

- set of detected errors keyed to the offending row/column (e.g. non-numeric value in a concentration column, missing primary ID, non-unique primary ID, missing-data rate above threshold)
- warnings (e.g. low-variance metabolite columns flagged as possible omission candidates)
- blocker status: pipeline halts until errors are corrected

## How to apply

Load the uploaded .csv or .xlsx table with Pandas, after the user designates which columns are metabolite concentration columns (via the drag-and-drop upload page's column-type designation). Scan each designated concentration column and flag any non-numeric value as an error. Run the companion checks in the same validation pass: flag levels of missing data that exceed Viime's default threshold within a group or across all samples, flag the lack of a primary ID, flag a non-unique primary ID, and warn on columns with excessively low variance (a likely candidate for omission). Errors must be corrected by the user — designating columns as primary ID, masked/hidden, factor, group, or metabolite concentration — before automatic data imputation (MCAR/MNAR classification) and the normalization → transformation → scaling pipeline can run.

## Related tools

- **Viime** (Hosts the upload-time validation: detects errors including non-numeric data in concentration columns during the drag-and-drop upload page's error-correction workflow) — girder/viime
- **Pandas** (Backend library that parses the raw uploaded .csv/.xlsx files before validation checks are applied) — girder/viime

## Examples

```
viime-cli create-tables && flask run  # then drag a .csv from samples/data into the upload page and designate concentration columns; validation flags non-numeric cells
```

## Evaluation signals

- A non-numeric token placed in a designated concentration column must surface as a detected error, not a silent failure.
- A table with a missing or duplicated primary ID must also surface an error (the check runs in the same pass).
- A group or all-sample missing-data rate above Viime's default threshold must be flagged.
- A metabolite column with excessively low variance must generate a warning (not an error) suggesting possible omission.
- No imputation or PCA output can be produced while any detected error remains uncorrected.

## Limitations

- Validation only catches structural/typing problems (non-numeric concentration data, missing/non-unique primary ID, threshold-exceeding missing data, low variance); it does not detect annotation or preprocessing errors — Viime's scope begins after preprocessing and annotation.
- Only .xlsx and .csv uploads are accepted by the drag-and-drop interface.
- The missing-data threshold is a default value; users cannot tune it from the described interface.

## Evidence

- [abstract] Errors detected at upload: "Errors that are detected include levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration data, the lack of a primary ID, and"
- [abstract] Error correction before imputation: "The user then is able to correct any errors, designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [abstract] Upload formats: "interactive drag and drop data upload interface which currently accepts .xlsx and .csv files"
- [abstract] Pandas parses raw files: "The backend leverages Pandas for raw file parsing and normalization."
- [other] Low-variance warning (Figure 2): [evidence span withheld — anchor status: not_found]
- [abstract] Viime scope after preprocessing: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
- [readme] Sample data directory: "you can use it to upload the CSV files found in the [sample data](samples/data) directory of this repository."
