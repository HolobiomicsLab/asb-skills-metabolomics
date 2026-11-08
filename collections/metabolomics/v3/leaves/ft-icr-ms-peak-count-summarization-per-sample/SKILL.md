---
name: ft-icr-ms-peak-count-summarization-per-sample
description: Use when a filtered FT-ICR MS peak table with assigned molecular formulas
  has been produced (e.g., by MetaboDirect's data pre-processing step) and you need
  per-sample counts and assignment-error diagnostics to validate data quality.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or a peak table that has not yet undergone molecular
  formula assignment and pre-processing filters (m/z, 13C isotopes, 0.5 ppm error)
- The goal is downstream chemodiversity, multivariate statistics, or transformation-network
  analysis rather than per-sample QC overview
- The data set was already filtered and summarized by the original authors (as with
  the S. fallax leachate data set, where no peak was filtered by MetaboDirect)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose data diagnostics step generates the per-sample peak/formula
    count plots and tables; runs with a single command line call
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Count total peaks and assigned molecular formulas per sample from the filtered
    peak table; export count tables as .csv
- name: Matplotlib
  role: Render bar plots of per-sample counts and faceted scatterplots of assignment
    error
- name: seaborn
  role: Bar-plot and faceted-scatterplot visualization of per-sample peak and formula
    counts
- name: NumPy
  role: Numerical computation on count and error (ppm) arrays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:71fdb0f50ca28adf54e29d8ec34f067c8c6edbedb9d50770635984081ffd879e
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# FT-ICR MS peak-count summarization per sample

## Summary

Data-diagnostics skill from the MetaboDirect pipeline that summarizes per-sample FT-ICR MS peak counts and molecular formula assignments as bar plots, .csv count tables, and faceted scatterplots of formula-assignment error (ppm). It provides a quick QC overview of the filtered peak table before deeper chemodiversity or statistical analysis.

## When to use

Use when a filtered FT-ICR MS peak table with assigned molecular formulas has been produced (e.g., by MetaboDirect's data pre-processing step) and you need per-sample counts and assignment-error diagnostics to validate data quality.

## When NOT to use

- Input is raw FT-ICR MS spectra or a peak table that has not yet undergone molecular formula assignment and pre-processing filters (m/z, 13C isotopes, 0.5 ppm error)
- The goal is downstream chemodiversity, multivariate statistics, or transformation-network analysis rather than per-sample QC overview
- The data set was already filtered and summarized by the original authors (as with the S. fallax leachate data set, where no peak was filtered by MetaboDirect)

## Inputs

- Filtered peak abundance table from MetaboDirect data pre-processing (peaks with assigned molecular formulas, error in ppm, sample columns)
- Per-sample metadata implicit in the peak table columns

## Outputs

- .csv tables of total peaks per sample
- .csv tables of assigned molecular formula counts per sample
- Bar plots of per-sample peak counts (Matplotlib/seaborn)
- Bar plots of per-sample formula-assignment counts (Matplotlib/seaborn)
- Faceted scatterplots of formula-assignment error distribution (ppm) per sample

## How to apply

Load the filtered peak table output from the MetaboDirect data pre-processing step (which has already applied m/z filtering, removal of 13C isotopic masses, and the 0.5 ppm formula-assignment error cutoff). Using pandas, count total peaks detected per sample and count peaks with assigned molecular formulas per sample; export these counts as .csv tables. Generate bar plots of the per-sample peak counts and per-sample formula-assignment counts with seaborn/Matplotlib. Compute the formula-assignment error (ppm) values from the filtered table and draw faceted scatterplots of error per sample. Write all diagnostic figures and tables to the pipeline output directory; these outputs are intended to give a quick overview of the data upon which more in-depth analysis (chemodiversity, multivariate statistics, transformation networks) can be built.

## Related tools

- **MetaboDirect** (Pipeline whose data diagnostics step generates the per-sample peak/formula count plots and tables; runs with a single command line call) — https://github.com/Coayala/MetaboDirect
- **pandas** (Count total peaks and assigned molecular formulas per sample from the filtered peak table; export count tables as .csv)
- **Matplotlib** (Render bar plots of per-sample counts and faceted scatterplots of assignment error)
- **seaborn** (Bar-plot and faceted-scatterplot visualization of per-sample peak and formula counts)
- **NumPy** (Numerical computation on count and error (ppm) arrays)

## Examples

```
import pandas as pd; counts = filtered_table.groupby('Sample').size(); counts.to_csv('peaks_per_sample.csv')
```

## Evaluation signals

- Every sample present in the filtered peak table appears in each diagnostic bar plot and .csv count table
- Per-sample formula-assignment counts never exceed total per-sample peak counts
- Formula-assignment error values plotted fall within the pre-processing cutoff (0.5 ppm) applied to the filtered table
- Each .csv count table matches counts recomputed from the filtered peak table (e.g., groupby-sample tallies)
- All diagnostic figures and tables are written to the pipeline output directory without missing files

## Limitations

- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis of the counted peaks
- Counts only provide a quick overview of the data, upon which more in-depth analysis must be built; they do not by themselves validate formula correctness
- Diagnostics reflect the state of the table after pre-processing filters; peaks removed by 13C-isotope filtering (~200 masses in the bacterium-phage data set) will not appear

## Evidence

- [intro] Pipeline structure places data diagnostics as second step: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics,"
- [abstract] Python stack used for the plotting and counting: [evidence span withheld — anchor status: not_found]
- [discussion] Purpose of the diagnostic overview: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
- [intro] Pre-processing filters applied before diagnostics: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline is command-line based for direct injection FT-ICR MS data: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
