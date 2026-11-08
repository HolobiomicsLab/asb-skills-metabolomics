---
name: metabolomics-table-csv-roundtrip
description: Use when the input is a metabolomics concentration table in .csv or .xlsx
  with mixed metadata and measurement columns, and it must be parsed, normalized,
  or transformed and exported as CSV for downstream statistical analysis.
when_to_use_negative:
- The input is raw instrument output (e.g. MS or NMR spectra) requiring preprocessing
  or annotation — Viime explicitly picks up the workflow after preprocessing and annotation
- The table is already normalized or the goal is downstream statistical analysis or
  visualization rather than table treatment
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Pandas
  role: parses the raw CSV/XLSX metabolite table and performs normalization before
    writing the result back to CSV
  repo: https://github.com/girder/viime
- name: Viime
  role: web application providing drag-and-drop CSV/XLSX upload, column type designation,
    and normalization options
  repo: https://github.com/girder/viime
- name: OpenCPU
  role: REST bridge between pandas data frames and the R viime package functions that
    accept CSV paths and return data frames
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:f895f48cfb51e08d7924c3bb70e453e64c15a727d7c19d887fd4f1e5c77547ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-table-csv-roundtrip

## Summary

Loads metabolite concentration tables from CSV/XLSX into pandas with sample identifiers, factor/group, and metabolite concentration columns intact, applies user-selected preprocessing (e.g. normalization), and writes the result back as CSV — the table roundtrip underpinning Viime's data-treatment workflow.

## When to use

Use when the input is a metabolomics concentration table in .csv or .xlsx with mixed metadata and measurement columns, and it must be parsed, normalized, or transformed and exported as CSV for downstream statistical analysis.

## When NOT to use

- The input is raw instrument output (e.g. MS or NMR spectra) requiring preprocessing or annotation — Viime explicitly picks up the workflow after preprocessing and annotation
- The table is already normalized or the goal is downstream statistical analysis or visualization rather than table treatment

## Inputs

- metabolite concentration table as .csv or .xlsx (sample rows, factor/group columns, metabolite concentration columns)
- normalization method parameter (reference sample, sum-to-fixed-value default 100, or weight/volume column)

## Outputs

- normalized concentration table written as a CSV file
- report of which normalization method was applied

## How to apply

Read the uploaded table with pandas, designating each column as the primary ID, masked/hidden, factor, group, or metabolite concentration type before any computation. Apply the user-selected normalization across only the metabolite concentration columns — dividing by a reference sample's metabolite sum, scaling each sample's sum to a fixed value (default 100), or normalizing by a weight/volume column — leaving metadata columns untouched. Persist the result by writing the treated data frame back to CSV; in the Viime backend this roundtrip is bridged to R processing functions via OpenCPU, where functions accept a CSV path and return a data frame. Validate that no metadata columns were altered and that the chosen normalization method is reported with the output.

## Related tools

- **Pandas** (parses the raw CSV/XLSX metabolite table and performs normalization before writing the result back to CSV) — https://github.com/girder/viime
- **Viime** (web application providing drag-and-drop CSV/XLSX upload, column type designation, and normalization options) — https://github.com/girder/viime
- **OpenCPU** (REST bridge between pandas data frames and the R viime package functions that accept CSV paths and return data frames)

## Examples

```
import pandas as pd; df = pd.read_csv('samples/data/table.csv'); mets = df.columns[3:]; df[mets] = df[mets].div(df[mets].sum(axis=1), axis=0) * 100; df.to_csv('normalized.csv', index=False)
```

## Evaluation signals

- Metadata columns (primary ID, factor, group) are byte-identical between input and output CSV
- Each sample row's metabolite sum equals the chosen fixed value (e.g. 100) after sum-scaling normalization
- Output parses cleanly as CSV with the same number of rows as the input table
- No non-numeric values remain in metabolite concentration columns (Viime flags non-numeric concentration data as an error)
- The normalization method applied is explicitly reported alongside the output file

## Limitations

- Table columns must first be validated — Viime detects errors such as missing data above a default threshold, non-numeric concentration data, and a missing or non-unique primary ID before treatment
- R processing functions in the viime package are work-in-progress and communicate with pandas via the opencpu.py bridge
- The web client must be running separately (yarn serve) even when the backend is launched via docker-compose

## Evidence

- [abstract] pandas parsing and normalization: "The backend leverages Pandas for raw file parsing and normalization."
- [abstract] accepted upload formats: "we have designed an interactive drag and drop data upload interface which currently accepts .xlsx and .csv files."
- [abstract] normalization is first treatment step: "The first step in this process is data normalization."
- [readme] R functions operate on CSV paths: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
