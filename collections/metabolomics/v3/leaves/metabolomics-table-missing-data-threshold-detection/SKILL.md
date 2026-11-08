---
name: metabolomics-table-missing-data-threshold-detection
description: Use when a freshly imported .csv or .xlsx metabolomics table must be
  checked for processing-readiness before imputation or normalization, especially
  when groups may contain too many missing values.
when_to_use_negative:
- The table has already passed validation and errors have been corrected — Viime automatically
  proceeds to imputation at that point, so re-validating would duplicate work.
- The input is raw instrument output (e.g., MS/NMR spectra) rather than a post-preprocessing,
  post-annotation metabolomics table; Viime's stated scope is to pick up the workflow
  after preprocessing and annotation.
- The goal is choosing imputation strategy (MCAR/MNAR) or normalization/transformation
  options rather than gating the dataset for processing.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3311
tools:
- name: Viime
  role: Application whose upload-time validation logic performs missing-data threshold
    checks, non-numeric concentration detection, primary ID checks, and low-variance
    warnings
  repo: girder/viime
- name: Pandas
  role: Used by Viime's backend to parse the raw .csv/.xlsx table so per-group missing
    rates and per-metabolite variance can be computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:f95440ae00375a843efb8c2bf06327d52e2db1e4e9bf95ec5aac3242433f0565
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3311
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolomics-table-missing-data-threshold-detection

## Summary

Upload-time validation used by Viime to decide whether an imported metabolomics table (.csv/.xlsx) is ready for processing, detecting threshold-exceeding missing data, non-numeric concentration values, and missing or non-unique primary IDs, while warning on low-variance metabolites.

## When to use

Use when a freshly imported .csv or .xlsx metabolomics table must be checked for processing-readiness before imputation or normalization, especially when groups may contain too many missing values.

## When NOT to use

- The table has already passed validation and errors have been corrected — Viime automatically proceeds to imputation at that point, so re-validating would duplicate work.
- The input is raw instrument output (e.g., MS/NMR spectra) rather than a post-preprocessing, post-annotation metabolomics table; Viime's stated scope is to pick up the workflow after preprocessing and annotation.
- The goal is choosing imputation strategy (MCAR/MNAR) or normalization/transformation options rather than gating the dataset for processing.

## Inputs

- .csv or .xlsx metabolomics table with concentration values, uploaded via Viime's drag-and-drop interface
- column type designations (primary ID, factor, group, metabolite concentration) for the imported table

## Outputs

- structured validation report of errors and warnings keyed by row/column (issue type, affected row/column, group/sample context)
- flagged groups or samples whose missing-data level exceeds the default threshold
- low-variance metabolite warnings suggesting possible omission

## How to apply

Load the uploaded .csv or .xlsx table with Pandas, which Viime's backend uses for raw file parsing, and identify the primary ID column, group/factor columns, and metabolite concentration columns. Compute per-group and all-sample missing-data rates and flag any group or sample set whose missing level exceeds Viime's default threshold. Scan concentration columns for non-numeric values and flag them as errors; verify a primary ID exists and that its values are unique. Additionally compute per-metabolite variance and emit warnings (not errors) for low-variance metabolites, which Viime flags for possible omission. Assemble all errors and warnings into a structured report keyed by row/column so the user can correct issues before the automatic imputation step runs.

## Related tools

- **Viime** (Application whose upload-time validation logic performs missing-data threshold checks, non-numeric concentration detection, primary ID checks, and low-variance warnings) — girder/viime
- **Pandas** (Used by Viime's backend to parse the raw .csv/.xlsx table so per-group missing rates and per-metabolite variance can be computed)

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv'); rates = df.isna().mean(); print(rates[rates > threshold])
```

## Evaluation signals

- The validation report distinguishes errors from warnings: non-unique/missing primary ID and non-numeric concentration values appear as errors, while low-variance metabolites appear as warnings for possible omission.
- Missing-data rates are evaluated both within each group and across all samples, and only levels exceeding the default threshold are flagged.
- A table lacking any primary ID column is rejected, and a table with duplicate primary ID values is rejected.
- After all flagged errors are corrected, the dataset proceeds to automatic imputation without re-raising the same errors.
- Report entries reference affected row/column and group/sample context so the user can locate and fix each issue in the original table.

## Limitations

- The missing-data threshold is a default value; users cannot rely on the article's text for a user-configurable threshold behavior.
- Low-variance metabolites are only warned about, not blocked — the decision to omit them rests with the user.
- Validation is defined for tabular .csv/.xlsx metabolomics tables only; the article's workflow starts after preprocessing and annotation, so upstream QC is out of scope.

## Evidence

- [abstract] errors include threshold-exceeding missing data and ID problems: "Errors that are detected include levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration data, the lack of a primary ID, and"
- [abstract] column designation enables error correction before imputation: "The user then is able to correct any errors, designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [abstract] imputation follows error correction: "Once errors are corrected, data imputation is automatically performed."
- [abstract] Pandas parses raw uploaded tables: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] sample CSV files can be exercised through the client: "Once the client is running, you can use it to upload the CSV files found in the [sample data](samples/data) directory of this repository."
