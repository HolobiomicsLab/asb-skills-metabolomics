---
name: metabolite-concentration-table-normalization
description: Use when a metabolite concentration table (.csv or .xlsx) contains samples
  with unequal total signal, weight, or volume and the values must be made consistent
  before data transformation, scaling, or PCA/group-comparison analysis.
when_to_use_negative:
- The table has already been normalized (e.g., per-sample sums are already constant)
  — re-normalizing can distort values; skip to the transformation step (Log10/Log2/square
  root/cube root).
- The data still contain uncorrected errors or missing values — Viime performs error
  correction and automatic MCAR/MNAR imputation before normalization.
- The goal is integrating multiple platforms' datasets (concatenative, mid-level,
  or multi-block fusion) rather than normalizing a single table.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3820
tools:
- name: Viime
  role: web-based application whose first data-treatment step is normalization, offering
    reference-sample, sum-scaling (default 100), and weight/volume options
  repo: https://github.com/girder/viime
- name: Pandas
  role: backend library used by Viime for raw file parsing and normalization of the
    concentration table
- name: R
  role: specialized data-processing algorithms, including preprocessing functions,
    are implemented in R packages and exposed through OpenCPU
- name: OpenCPU
  role: internal REST service providing the R–Python integration for CSV file processing
    functions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:20a807e9719fd5058f1dbe1d6d6fc48b2f0d14c61f5157bdfc0351d8c785068d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3820
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-concentration-table-normalization

## Summary

Normalizes metabolite concentration values in a per-sample metabolomics table so samples are comparable, using Viime-style options: normalization to a reference sample's metabolite sum, scaling each sample's sum to a fixed value (default 100), or normalization by a sample weight/volume column. This is the first data-treatment step Viime applies before transformation, scaling, and statistical analysis.

## When to use

Use when a metabolite concentration table (.csv or .xlsx) contains samples with unequal total signal, weight, or volume and the values must be made consistent before data transformation, scaling, or PCA/group-comparison analysis.

## When NOT to use

- The table has already been normalized (e.g., per-sample sums are already constant) — re-normalizing can distort values; skip to the transformation step (Log10/Log2/square root/cube root).
- The data still contain uncorrected errors or missing values — Viime performs error correction and automatic MCAR/MNAR imputation before normalization.
- The goal is integrating multiple platforms' datasets (concatenative, mid-level, or multi-block fusion) rather than normalizing a single table.

## Inputs

- metabolite concentration table in .csv or .xlsx format (drag-and-drop uploaded), with sample rows and metabolite concentration columns
- column designation metadata (primary ID, masked/hidden, factor, group, metabolite concentration) from the table-structure designation step
- normalization method parameter: reference sample sum, fixed sum value (default 100), or sample weight/volume column

## Outputs

- normalized metabolite concentration table with consistent per-sample totals, written back out as a CSV file
- record of which normalization method was applied

## How to apply

Load the uploaded metabolite concentration table (CSV/XLSX) with pandas, retaining sample identifiers, factor/group columns, and metabolite concentration columns, and leaving metadata columns untouched. Choose the normalization method as a user-selected parameter: (1) divide each sample by a reference sample's metabolite sum, (2) scale each sample's metabolite sum to a fixed value with default 100, or (3) normalize by a sample weight or volume column. Apply the selected method uniformly across all metabolite concentration columns, leaving primary ID, masked/hidden, factor, and group columns unmodified. In Viime this step runs first in the data-treatment sequence — before transformation (Log10, Log2, square root, cube root) and scaling (autoscaling, Pareto, range, vast, level) — with interactive PCA used to examine whether the treatment choices separate or align groups. Write the normalized dataset back out as a CSV and report which normalization method was applied; in Viime's backend, processing functions accept a path to a CSV file and return a data frame, with R-implemented functions exposed via the OpenCPU REST service.

## Related tools

- **Viime** (web-based application whose first data-treatment step is normalization, offering reference-sample, sum-scaling (default 100), and weight/volume options) — https://github.com/girder/viime
- **Pandas** (backend library used by Viime for raw file parsing and normalization of the concentration table)
- **R** (specialized data-processing algorithms, including preprocessing functions, are implemented in R packages and exposed through OpenCPU)
- **OpenCPU** (internal REST service providing the R–Python integration for CSV file processing functions)

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv', index_col=0); df_norm = df.div(df.sum(axis=1), axis=0) * 100
```

## Evaluation signals

- After sum-scaling normalization, each sample's metabolite concentration columns sum to the fixed value (default 100).
- After reference-sample normalization, every sample's metabolite sum is expressed relative to the reference sample's metabolite sum.
- Metadata columns (primary ID, factor, group, masked/hidden) are byte-identical to the input table.
- Row (sample) and metabolite-column counts are preserved; no samples or metabolites were dropped by normalization.
- The output CSV records which normalization method was applied.

## Limitations

- Normalization is user-selected, so an inappropriate choice (e.g., weight/volume when the weight column is missing) must be guarded against; Viime's error detection covers missing data thresholds, non-numeric concentration data, missing/non-unique primary IDs, and low-variance metabolites before treatment.
- In the repository, the R processing backend is described as work-in-progress; OpenCPU-based processing requires spinning up the viime Docker image (docker run -p 8004:8004 viime).
- The web client still needs to be started separately (yarn serve) even under docker-compose; the R package functions only handle CSV file paths.

## Evidence

- [abstract] Normalization precedes transformation and scaling in Viime's data-treatment workflow: "The first step in this process is data normalization."
- [abstract] Pandas performs raw file parsing and normalization in Viime's backend: "The backend leverages Pandas for raw file parsing and normalization."
- [abstract] Normalization options follow the data-treatment sequence before transformation and scaling: "The options in this step are Log10, Log2, square root and cube root."
- [readme] Backend processing functions operate on CSV files and return data frames: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] R processing functions are exposed to the API server via a custom package in the OpenCPU docker image: "This installation includes a custom R package called
[viime](devops/viime), which contains all functions that are exposed to
the API server for CSV file processing."
