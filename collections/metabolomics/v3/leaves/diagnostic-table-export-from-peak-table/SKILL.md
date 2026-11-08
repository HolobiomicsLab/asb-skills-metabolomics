---
name: diagnostic-table-export-from-peak-table
description: Use when a filtered FT-ICR MS peak/abundance table with assigned molecular
  formulas is available and you need per-sample counts (peaks, formulas) and assignment-error
  diagnostics as .csv tables and plots.
when_to_use_negative:
- Input is raw FT-ICR MS spectra that have not yet undergone molecular formula assignment
  — MetaboDirect expects peak abundance and formula data produced after initial processing
- The goal is a specific chemodiversity, multivariate-statistics, Van Krevelen, or
  transformation-network analysis rather than a per-sample data-quality overview
- The peak table has not been filtered (e.g., unfiltered isotopic 13C masses or masses
  failing the 0.5 ppm error threshold remain), which will inflate the diagnostic counts
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose data-diagnostics step generates the per-sample peak/formula
    count plots, .csv tables, and error scatterplots
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Counting total peaks and assigned formulas per sample from the filtered peak
    table
- name: seaborn
  role: Rendering bar plots of per-sample counts and faceted scatterplots of formula-assignment
    error
- name: Matplotlib
  role: Plotting backend for the diagnostic bar plots and scatterplots
- name: NumPy
  role: Numerical support for computing per-sample summary statistics from the peak
    table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:b0895298f8e1f9f93f286b322b85201a89b75d793ea0ba4ff85627e0cbab623a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Diagnostic table export from peak table

## Summary

Generates per-sample data-diagnostics summaries — bar plots and .csv tables of total peak counts and molecular formula assignments, plus faceted scatterplots of formula-assignment error (ppm) — from the filtered FT-ICR MS peak table, as implemented in step (ii) of the MetaboDirect pipeline.

## When to use

Use when a filtered FT-ICR MS peak/abundance table with assigned molecular formulas is available and you need per-sample counts (peaks, formulas) and assignment-error diagnostics as .csv tables and plots.

## When NOT to use

- Input is raw FT-ICR MS spectra that have not yet undergone molecular formula assignment — MetaboDirect expects peak abundance and formula data produced after initial processing
- The goal is a specific chemodiversity, multivariate-statistics, Van Krevelen, or transformation-network analysis rather than a per-sample data-quality overview
- The peak table has not been filtered (e.g., unfiltered isotopic 13C masses or masses failing the 0.5 ppm error threshold remain), which will inflate the diagnostic counts

## Inputs

- Filtered FT-ICR MS peak abundance table with assigned molecular formulas (output of MetaboDirect data pre-processing)
- Per-sample metadata/column structure implicit in the peak table (samples as columns)

## Outputs

- .csv table of total peaks detected per sample
- .csv table of molecular formula assignments per sample
- Bar plot of per-sample peak counts (seaborn/Matplotlib figure)
- Bar plot of per-sample formula-assignment counts (seaborn/Matplotlib figure)
- Faceted scatterplots of formula-assignment error (ppm) distribution per sample

## How to apply

Load the filtered peak table produced by the MetaboDirect data pre-processing step, which filters peaks by m/z, 13C isotopic presence, formula-assignment error (0.5 ppm), and a user-defined minimum number of samples. Using pandas, count total peaks detected per sample and molecular formulas assigned per sample, then export these counts as .csv tables. Render bar plots of both per-sample count summaries with seaborn/Matplotlib, and plot faceted scatterplots of formula-assignment error in ppm per sample. Write all figures and tables to the output directory; because MetaboDirect is designed to run as a single-line command producing all analyses automatically, these diagnostics can also be regenerated via the pipeline itself. Rationale: the diagnostics step gives a quick, per-sample overview of data quality and formula coverage before deeper chemodiversity, statistical, or transformation-network analyses.

## Related tools

- **MetaboDirect** (Pipeline whose data-diagnostics step generates the per-sample peak/formula count plots, .csv tables, and error scatterplots) — https://github.com/Coayala/MetaboDirect
- **pandas** (Counting total peaks and assigned formulas per sample from the filtered peak table)
- **seaborn** (Rendering bar plots of per-sample counts and faceted scatterplots of formula-assignment error)
- **Matplotlib** (Plotting backend for the diagnostic bar plots and scatterplots)
- **NumPy** (Numerical support for computing per-sample summary statistics from the peak table)

## Examples

```
metabodirect -i filtered_peak_table.csv -s metadata.csv -o out_dir  # data diagnostics (counts, .csv tables, error plots) run automatically
```

## Evaluation signals

- Every sample present in the input filtered peak table appears in each exported diagnostic .csv table (no samples dropped).
- Per-sample peak counts in the .csv match a direct pandas value count on the filtered peak table.
- Per-sample formula-assignment counts match the number of rows with a non-missing assigned molecular formula in the filtered table.
- Error-distribution scatterplot ppm values reconcile with the formula-assignment error column, whose values passed the 0.5 ppm filter during pre-processing.
- All expected figure and .csv files are present in the output directory after the run.

## Limitations

- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream analysis, so diagnostics reflect only what direct-injection FT-ICR MS resolved.
- Diagnostic counts are only as good as the upstream filtering step; e.g., ~200 masses containing isotopic carbon were filtered out of the bacterium-phage data set, and counts change accordingly.
- If the input data set was previously filtered by other authors (as with the S. fallax leachate data), diagnostics will report the pre-filtered counts without further trimming.

## Evidence

- [results] per-sample count summaries: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis"
- [intro] filtering thresholds: [evidence span withheld — anchor status: not_found]
- [intro] single-line run design: "designed to run using a single line of code to automatically produce all the analyses, figures, and tables described in its documentation without requiring long computing times"
- [discussion] overview purpose: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
- [abstract] stack details: [evidence span withheld — anchor status: not_found]
