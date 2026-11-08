---
name: reference-sample-sum-scaling
description: Use when an uploaded metabolite concentration table (.csv or .xlsx) has
  per-sample totals that differ due to loading or volume differences, and consistent
  per-sample measurement values are needed before Viime's transformation, scaling,
  and PCA steps.
when_to_use_negative:
- The input is not a per-sample metabolite concentration table with designated primary
  ID/group/factor columns (e.g., already-fused multi-platform data or raw instrument
  output).
- Normalization has already been applied and the next data-treatment step (Log10/Log2/square/cube-root
  transformation or autoscaling/Pareto/range/vast/level scaling) is what is needed.
- The goal is cross-platform dataset integration rather than within-table normalization
  — use Viime's dataset-merge fusion algorithms instead.
tools:
- name: Pandas
  role: parsing the raw CSV/XLSX metabolite table and applying normalization across
    metabolite concentration columns while metadata columns are left untouched
  repo: girder/viime
- name: Viime
  role: web application exposing the normalization step of the data-treatment process
    as a user-selectable option after data upload and imputation
  repo: girder/viime
- name: OpenCPU
  role: internal REST service providing the R-python integration through which preprocessing
    steps such as normalization are executed
  repo: girder/viime
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:315b9ca2b04d83f8916a8c79126ff75df0d27771a48d40aa59ac31232d502706
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - Viime
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# reference-sample-sum-scaling

## Summary

Per-sample normalization of metabolite concentration tables (CSV/XLSX) in Viime's data-treatment chain: scale each sample's metabolite values by a reference sample's metabolite sum, to a fixed sum (default 100), or by a sample weight/volume column, so per-sample totals are consistent before transformation, scaling, and PCA.

## When to use

Use when an uploaded metabolite concentration table (.csv or .xlsx) has per-sample totals that differ due to loading or volume differences, and consistent per-sample measurement values are needed before Viime's transformation, scaling, and PCA steps.

## When NOT to use

- The input is not a per-sample metabolite concentration table with designated primary ID/group/factor columns (e.g., already-fused multi-platform data or raw instrument output).
- Normalization has already been applied and the next data-treatment step (Log10/Log2/square/cube-root transformation or autoscaling/Pareto/range/vast/level scaling) is what is needed.
- The goal is cross-platform dataset integration rather than within-table normalization — use Viime's dataset-merge fusion algorithms instead.

## Inputs

- metabolite concentration table as .csv or .xlsx (metabolite concentration columns plus sample ID, group/factor metadata columns)
- selected normalization method (reference sample, sum-to-fixed-value, weight/volume column, or min-max)
- reference sample row (for reference-sample mode) or sample weight/volume column

## Outputs

- normalized dataset written back out as a CSV file
- report of which normalization method was applied

## How to apply

Load the uploaded metabolite concentration table (.csv/.xlsx) with pandas, retaining sample identifiers, group/factor columns, and metabolite concentration columns. Choose the normalization method: divide each sample by a reference sample's metabolite sum, scale each sample's metabolite sum to a fixed value (default 100), min-max normalize, or normalize by a sample's weight/volume column — the user selects the method. Apply the chosen normalization across all metabolite concentration columns while leaving metadata columns (primary ID, factor, group, masked/hidden) untouched. Write the normalized dataset back out as a CSV file and report which normalization method was applied. In Viime this is the first step of the data-treatment process, preceding imputation-driven error correction, log/square/cube-root transformation, and autoscaling/Pareto/range/vast/level scaling; specialized processing steps are implemented as R package functions reachable through OpenCPU, with pandas handling raw file parsing and normalization.

## Related tools

- **Pandas** (parsing the raw CSV/XLSX metabolite table and applying normalization across metabolite concentration columns while metadata columns are left untouched) — girder/viime
- **Viime** (web application exposing the normalization step of the data-treatment process as a user-selectable option after data upload and imputation) — girder/viime
- **OpenCPU** (internal REST service providing the R-python integration through which preprocessing steps such as normalization are executed) — girder/viime

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv'); mets = df.columns[4:]; df[mets] = df[mets].div(df.loc[df['sample']=='ref', mets].sum(axis=1)[0], axis=1); df.to_csv('normalized.csv')
```

## Evaluation signals

- Metadata columns (primary ID, factor, group, masked/hidden) are byte-identical before and after normalization; only metabolite concentration columns changed.
- Under sum scaling, every sample's metabolite sum equals the target value (default 100) after the step.
- Under reference-sample normalization, each sample's metabolite sum equals the reference sample's metabolite sum.
- Under weight/volume normalization, each sample's values equal the raw values divided by the corresponding weight/volume column entry.
- Output is a valid CSV with the same row/column count as the input table, and the applied method is reported.

## Limitations

- The method must be selected by the user; Viime does not choose the normalization strategy automatically.
- Normalization is the first data-treatment step and runs after error correction and automatic imputation, so it cannot fix upstream issues like non-numeric concentration data or a missing primary ID.
- R-implemented processing functions are exposed via an OpenCPU docker service, so the normalization backend requires that container to be running (e.g., docker run -p 8004:8004 viime).

## Evidence

- [other] normalizing to a reference sample's metabolite sum, scaling each sample's sum to a set value (default 100), or using a column of sample weights or volumes: [evidence span withheld — anchor status: not_found]
- [abstract] The first step in this process is data normalization.: "The first step in this process is data normalization."
- [abstract] The backend leverages Pandas for raw file parsing and normalization.: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] Functions contained in this package should accept a path to a CSV file as an argument and return a data frame.: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
