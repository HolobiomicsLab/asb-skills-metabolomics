---
name: weight-volume-column-normalization
description: Use when a loaded metabolite concentration table (CSV/XLSX) includes
  a column of sample weights or volumes and per-sample concentration values must be
  made consistent by dividing through that column, leaving metadata columns untouched.
when_to_use_negative:
- The table has no weight/volume column — use Viime's reference-sample or sum-scaling
  normalization instead
- Data is post-normalization and needs Log10/Log2/square-root/cube-root transformation
  or autoscaling/Pareto/range/vast/level scaling, which are separate Viime steps
- Missing values are present and uncorrected — Viime performs imputation with MNAR/MNAR
  classification before normalization, so error correction and imputation should come
  first
edam_operation: http://edamontology.org/operation_2928
tools:
- name: Pandas
  role: Parses the raw CSV/XLSX concentration table and applies the column-wise division
    by the weight/volume column, per Viime's backend which 'leverages Pandas for raw
    file parsing and normalization'
- name: Viime
  role: Web application whose user-configurable preprocessing pipeline exposes the
    weight/volume normalization option before statistical analysis
  repo: https://github.com/girder/viime
- name: R
  role: Hosts the viime R package whose processing functions accept a path to a CSV
    file, perform the normalization, and return a data frame via the OpenCPU backend
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:a8002a7c1f1decfdd5f42c5eff9a42b6c490abd2168c459aef481dd11e6e832f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# weight-volume-column-normalization

## Summary

Normalizes metabolomics sample measurements by dividing each metabolite concentration column by a per-sample weight or volume column, one of Viime's user-selectable normalization options that makes per-sample values comparable before transformation, scaling, and statistical analysis.

## When to use

Use when a loaded metabolite concentration table (CSV/XLSX) includes a column of sample weights or volumes and per-sample concentration values must be made consistent by dividing through that column, leaving metadata columns untouched.

## When NOT to use

- The table has no weight/volume column — use Viime's reference-sample or sum-scaling normalization instead
- Data is post-normalization and needs Log10/Log2/square-root/cube-root transformation or autoscaling/Pareto/range/vast/level scaling, which are separate Viime steps
- Missing values are present and uncorrected — Viime performs imputation with MNAR/MNAR classification before normalization, so error correction and imputation should come first

## Inputs

- Metabolite concentration table as CSV or XLSX with sample identifiers, group/factor columns, metabolite concentration columns, and a sample weight or volume column

## Outputs

- Normalized dataset written back out as a CSV file with concentration columns divided by the weight/volume column and metadata columns unchanged
- Report of which normalization method was applied

## How to apply

Load the uploaded metabolite concentration table (CSV/XLSX) with pandas, retaining sample identifiers, group/factor columns, and metabolite concentration columns. Select 'weight/volume normalization' as the normalization method parameter, distinguishing it from Viime's alternatives: dividing by a reference sample's metabolite sum, or scaling each sample's metabolite sum to a fixed value (default 100). Apply the selected normalization across all metabolite concentration columns while leaving metadata (primary ID, masked/hidden, factor, group) columns untouched, since Viime lets the user designate any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column. Write the normalized dataset back out as a CSV and report which normalization method was applied; in Viime the processing is exposed through R functions (OpenCPU backend) that accept a CSV path and return a data frame. Afterward, examine the interactive PCA score and loadings plots Viime provides to confirm the treatment improved group comparability before proceeding to Log10/Log2/square-root/cube-root transformation and scaling (autoscaling, Pareto, range, vast, level).

## Related tools

- **Pandas** (Parses the raw CSV/XLSX concentration table and applies the column-wise division by the weight/volume column, per Viime's backend which 'leverages Pandas for raw file parsing and normalization')
- **Viime** (Web application whose user-configurable preprocessing pipeline exposes the weight/volume normalization option before statistical analysis) — https://github.com/girder/viime
- **R** (Hosts the viime R package whose processing functions accept a path to a CSV file, perform the normalization, and return a data frame via the OpenCPU backend) — https://github.com/girder/viime

## Examples

```
import pandas as pd; df = pd.read_csv('concentrations.csv'); df[metabolite_cols] = df[metabolite_cols].div(df['weight'], axis=0); df.to_csv('normalized.csv', index=False)
```

## Evaluation signals

- Each normalized concentration equals the original value divided by that sample's weight/volume entry (spot-check cells against the input CSV)
- Metadata columns (primary ID, factor, group, masked/hidden) are byte-identical to the input table
- Output is valid CSV that the viime R processing functions can round-trip (function accepts a CSV path and returns a data frame)
- Interactive PCA score/loadings plots show improved between-group comparability relative to the pre-normalization table
- The applied normalization method (weight/volume) is explicitly reported alongside the output

## Limitations

- Normalization is only valid if the weight/volume column is numeric and complete; Viime flags non-numeric concentration data and missing primary IDs as errors to correct first
- Normalization is one step in Viime's data treatment pipeline and is assessed jointly with transformation and scaling via PCA; it does not substitute for imputation or downstream scaling
- The R processing backend is described as work-in-progress in the repository README, and OpenCPU must be running (e.g. via the devops Docker container) for R-implemented steps

## Evidence

- [abstract] Normalization options: "The first step in this process is data normalization."
- [full_text] Normalization alternatives in Viime: [evidence span withheld — anchor status: not_found]
- [abstract] Column designation: "designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [readme] R functions for CSV processing: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [abstract] Pandas backend role: "The backend leverages Pandas for raw file parsing and normalization."
