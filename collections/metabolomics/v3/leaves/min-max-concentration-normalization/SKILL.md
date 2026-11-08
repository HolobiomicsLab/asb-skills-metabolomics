---
name: min-max-concentration-normalization
description: Use when a metabolite concentration table (CSV/XLSX) has columns designated
  as metabolite concentrations in Viime and per-sample values need range rescaling
  so all metabolites share a comparable scale prior to transformation, scaling, or
  PCA inspection.
when_to_use_negative:
- The table still contains uncorrected errors or missing values — Viime performs error
  detection and automatic imputation before normalization, so normalize only after
  imputation.
- The intended normalization is reference-sample-sum, fixed-sum (default 100), or
  weight/volume based — pick that specific method instead; Viime lets the user select
  among them.
- The goal is post-normalization treatment (Log10/Log2/square-root/cube-root transformation
  or autoscaling/Pareto/range/vast/level scaling) — those are distinct downstream
  steps, not this one.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Viime
  role: web application whose normalization step offers min-max alongside reference-sample,
    sum-scaling, and weight/volume options; user-configurable, R-implemented preprocessing
    before statistical analysis
  repo: girder/viime
- name: Pandas
  role: leverages raw file parsing and normalization of the concentration table in
    the Viime backend
- name: R
  role: viime R package functions exposed via OpenCPU perform CSV processing (accept
    a CSV path, return a data frame), including preprocessing steps such as normalization
- name: OpenCPU
  role: internal REST service providing the R–Python integration through which the
    R normalization functions are called
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:c2d45f124d8c3d7973af426e29c9bc1feec84445764a19e0b00eed4bca6becde
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# min-max-concentration-normalization

## Summary

Rescales each metabolite concentration column of a sample-by-metabolite table (CSV/XLSX) to a common [0,1] range via min-max normalization, one of the user-selectable normalization options in Viime's data-treatment pipeline, applied before transformation and scaling.

## When to use

Use when a metabolite concentration table (CSV/XLSX) has columns designated as metabolite concentrations in Viime and per-sample values need range rescaling so all metabolites share a comparable scale prior to transformation, scaling, or PCA inspection.

## When NOT to use

- The table still contains uncorrected errors or missing values — Viime performs error detection and automatic imputation before normalization, so normalize only after imputation.
- The intended normalization is reference-sample-sum, fixed-sum (default 100), or weight/volume based — pick that specific method instead; Viime lets the user select among them.
- The goal is post-normalization treatment (Log10/Log2/square-root/cube-root transformation or autoscaling/Pareto/range/vast/level scaling) — those are distinct downstream steps, not this one.

## Inputs

- sample-by-metabolite concentration table uploaded as .csv or .xlsx with designated primary ID, factor, group, and metabolite concentration columns
- user-selected normalization method parameter (min-max)

## Outputs

- normalized CSV in which each metabolite concentration column is min-max scaled to [0,1]
- report of the normalization method applied

## How to apply

Load the uploaded sample-by-metabolite table with pandas, retaining the columns the user designated as primary ID, masked/hidden, factor, group, or metabolite concentration. Choose min-max normalization from Viime's normalization options (the alternatives being division by a reference sample's metabolite sum, scaling each sample's sum to a fixed value such as the default 100, or normalization by a weight/volume column). Apply min-max rescaling — (value − column minimum) / (column maximum − column minimum) — across all metabolite concentration columns only, leaving metadata columns untouched. In Viime this normalization is the first step of the data-treatment process, followed by transformation (Log10, Log2, square root, cube root) and scaling (autoscaling, Pareto, range, vast, level), with interactive PCA used to judge whether the treatment is adequate. Viime's backend leverages Pandas for parsing and normalization, and normalization may be executed by R functions in the viime package exposed through OpenCPU, which accept a CSV path and return a data frame. Write the normalized dataset back out as CSV and report which normalization method was applied.

## Related tools

- **Viime** (web application whose normalization step offers min-max alongside reference-sample, sum-scaling, and weight/volume options; user-configurable, R-implemented preprocessing before statistical analysis) — girder/viime
- **Pandas** (leverages raw file parsing and normalization of the concentration table in the Viime backend)
- **R** (viime R package functions exposed via OpenCPU perform CSV processing (accept a CSV path, return a data frame), including preprocessing steps such as normalization)
- **OpenCPU** (internal REST service providing the R–Python integration through which the R normalization functions are called)

## Examples

```
df = pd.read_csv('concentrations.csv'); meta = df[['sample_id','group']]; conc = df.drop(columns=meta.columns); norm = (conc - conc.min()) / (conc.max() - conc.min()); pd.concat([meta, norm], axis=1).to_csv('normalized.csv', index=False)
```

## Evaluation signals

- Every metabolite concentration column in the output has minimum 0 and maximum 1 (all values within [0,1]).
- Metadata columns (primary ID, factor, group, masked/hidden) are byte-identical to the input — only concentration columns changed.
- The number of samples (rows) and metabolites (columns) is unchanged, confirming row-wise/column-wise rescaling rather than filtering.
- The output CSV is written and the applied normalization method (min-max) is reported.
- Downstream PCA score/loadings plots in Viime show the expected scale-driven change relative to the untreated data.

## Limitations

- The Viime abstract lists reference-sample, sum-scaling, and weight/volume as the described normalization options; min-max is one of the selectable methods in the workflow, so confirm it is exposed in the version in use.
- Normalization only runs after Viime's error correction and automatic imputation; tables with non-numeric concentration data, missing primary IDs, or missing data exceeding the default threshold will be flagged as errors first.
- Min-max rescaling, like all Viime data treatment, should be validated with the interactive PCA score and loadings plots to confirm group similarity/dissimilarity structure is preserved.

## Evidence

- [abstract] data normalization is the first data-treatment step in Viime: "The first step in this process is data normalization."
- [abstract] Pandas handles file parsing and normalization in the backend: "The backend leverages Pandas for raw file parsing and normalization."
- [abstract] users designate column roles including metabolite concentration before treatment: "The user then is able to correct any errors, designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [readme] R package functions process CSV files and return data frames via OpenCPU: "Functions contained in this package
should accept a path to a CSV file as an argument and return a data frame."
