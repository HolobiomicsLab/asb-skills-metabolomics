---
name: primary-id-uniqueness-check
description: Use when a freshly uploaded .csv or .xlsx metabolomics table must be
  validated for threshold-exceeding missing data, non-numeric concentration entries,
  or a missing/non-unique primary ID before imputation and normalization.
when_to_use_negative:
- The table has already passed validation and errors were corrected — at that point
  Viime automatically performs imputation, so re-running validation is redundant
- The input is raw instrument output (e.g. spectra) rather than a preprocessed, annotated
  concentration table — Viime's workflow picks up only after preprocessing and annotation
- The goal is integrating multiple validated datasets via concatenative, mid-level,
  or multi-block fusion — that is the dataset-merge step, not per-table validation
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Pandas
  role: Parses the raw .csv/.xlsx table and supports the missing-rate, numeric-type,
    ID-uniqueness, and variance checks
  repo: https://github.com/girder/viime
- name: Viime
  role: Host application whose upload-time validation detects the errors and warnings;
    users correct flagged issues interactively by re-designating column roles
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:486344fb5656bb3a2dd8fe39b833749ef61c2043aeb02d5b67a2e6e07c0b5785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# primary-id-uniqueness-check

## Summary

A tabular quality-validation step from the Viime metabolomics workflow that scans an uploaded .csv or .xlsx concentration table for blocking errors (missing data beyond a default threshold, non-numeric concentration values, a missing or non-unique primary ID) and for warnings such as low-variance metabolites, before imputation is allowed to proceed.

## When to use

Use when a freshly uploaded .csv or .xlsx metabolomics table must be validated for threshold-exceeding missing data, non-numeric concentration entries, or a missing/non-unique primary ID before imputation and normalization.

## When NOT to use

- The table has already passed validation and errors were corrected — at that point Viime automatically performs imputation, so re-running validation is redundant
- The input is raw instrument output (e.g. spectra) rather than a preprocessed, annotated concentration table — Viime's workflow picks up only after preprocessing and annotation
- The goal is integrating multiple validated datasets via concatenative, mid-level, or multi-block fusion — that is the dataset-merge step, not per-table validation

## Inputs

- Uploaded metabolomics table in .csv or .xlsx format with designated primary ID, factor/group, and metabolite concentration columns

## Outputs

- Structured validation report listing error types (threshold-exceeding missing data, non-numeric concentration values, missing primary ID, non-unique primary ID) keyed by affected row/column with group/sample context
- Warnings for low-variance metabolites flagged as possible omissions

## How to apply

Load the table with Pandas (as Viime's backend does for raw file parsing), identifying the primary ID column, group/factor columns, and metabolite concentration columns. Compute per-group and all-sample missing-data rates and raise an error wherever missingness exceeds Viime's default threshold within a group or across all samples; scan concentration columns for non-numeric values; and verify the primary ID column exists and its values are unique — flagging either failure as an error. Separately compute per-metabolite variance and emit warnings (not errors) for low-variance metabolites, which Viime surfaces as candidates for omission. Assemble errors and warnings into a structured report keyed by row/column with issue type and group/sample context; only after the user corrects the errors does Viime proceed to automatic MCAR/MNAR imputation.

## Related tools

- **Pandas** (Parses the raw .csv/.xlsx table and supports the missing-rate, numeric-type, ID-uniqueness, and variance checks) — https://github.com/girder/viime
- **Viime** (Host application whose upload-time validation detects the errors and warnings; users correct flagged issues interactively by re-designating column roles) — https://github.com/girder/viime

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv'); assert df['primary_id'].is_unique, 'non-unique primary ID'
```

## Evaluation signals

- The report enumerates exactly the four Viime error categories: missing data above the default threshold (per group and across all samples), non-numeric concentration data, lack of a primary ID, and non-uniqueness of the primary ID
- Every flagged issue carries row/column and group/sample context so the user can locate and correct it
- A known-unique ID column produces no non-uniqueness error; deleting the ID column or duplicating an ID triggers the corresponding error
- Low-variance metabolites appear as warnings (candidates for omission), not as blocking errors
- Validation runs before imputation: no imputed values exist in the output report's table state

## Limitations

- The missing-data threshold is Viime's fixed default; the article does not document a user-configurable cutoff
- Low-variance flags are heuristic warnings, not errors — a flagged metabolite may still be legitimately retained
- The CSV-processing backend path in the repository (viime/opencpu.py bridging Pandas and R data frames) is described as work-in-progress in the README

## Evidence

- [abstract] Detected error types enumerated by Viime's validation: "Errors that are detected include levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration data, the lack of a primary ID, and"
- [abstract] User correction of flagged errors by re-designating column roles: "The user then is able to correct any errors, designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [abstract] Validation gates the automatic imputation step: "Once errors are corrected, data imputation is automatically performed."
- [readme] Sample CSV data available for exercising upload and validation: "Once the client is running, you can use it to upload the CSV files found in the [sample data](samples/data) directory of this repository."
