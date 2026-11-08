---
name: ms1-feature-table-qc-summarization
description: Use when a pre-processed LC-MS/MS peak/feature table (e.g., Progenesis
  export such as PTY087I2) plus sample metadata must be QC-filtered and summarized
  into per-filter pass/fail ion counts before downstream analysis.
when_to_use_negative:
- Input is raw tandem MS/MS data or MS2-level fragmentation spectra rather than a
  pre-processed MS1 feature table from a peak-picking tool like Progenesis
- Data have no technical-replicate or blank-sample structure, making filter_cv() and
  filter_group() group definitions inapplicable
- The goal is analysis of fragment-ion MS2 spectra themselves rather than removal
  of in-source fragment ions from an MS1 abundance table
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: Core package providing import_data(), the independent filters (filter_mispicked_ions,
    filter_group, filter_cv, filter_insource_ions), filter_summary(), and extractors
    get_peak_table(), get_metadata(), get_group_averages(), get_cv_data()
  repo: https://github.com/mums2/mpactr
- name: R
  role: Implementation language and runtime for the mpactr QC pipeline; installable
    from CRAN or GitHub
- name: ggplot
  role: Used with plotly to create interactive plots of input features and the filters
    they failed
- name: plotly
  role: Provides interactivity for the ggplot filter-failure plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:93629f545501051caa02f23d1caa122ca533d61e8058d55a3c265886ec437b47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - data.table
  - ggplot
  - mpactr
  - plotly
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ms1-feature-table-qc-summarization

## Summary

A skill for quality-controlling and summarizing MS1 feature tables from LC-MS/MS pre-processing using mpactR, which automates correction of mispicked peaks, blank contamination, non-reproducible ions, and in-source fragments for microbiological samples.

## When to use

Use when a pre-processed LC-MS/MS peak/feature table (e.g., Progenesis export such as PTY087I2) plus sample metadata must be QC-filtered and summarized into per-filter pass/fail ion counts before downstream analysis.

## When NOT to use

- Input is raw tandem MS/MS data or MS2-level fragmentation spectra rather than a pre-processed MS1 feature table from a peak-picking tool like Progenesis
- Data have no technical-replicate or blank-sample structure, making filter_cv() and filter_group() group definitions inapplicable
- The goal is analysis of fragment-ion MS2 spectra themselves rather than removal of in-source fragment ions from an MS1 abundance table

## Inputs

- Progenesis peak table (e.g., dataset PTY087I2: 38 samples spanning solvent blanks, media blanks, and Streptomyces sp. PTY08712 grown with (250um_Ce) and without (0um_Ce) cerium)
- Combined sample list / extract metadata table
- User-defined parameters: cluster_threshold (e.g., 0.95) for in-source ion clustering, abundance thresholds for background components

## Outputs

- Filtered MS1 peak/feature table (via get_peak_table())
- Per-filter summaries (e.g., filter_summary(..., 'insource')) listing failed and passed ions
- Group-average abundance table (get_group_averages())
- CV table for technical-replicate reproducibility (get_cv_data())
- CSV exports of the filtered feature table and filter summaries
- Interactive ggplot/plotly plots of input features and the filters they failed

## How to apply

Import the Progenesis peak table and combined sample-list/extract metadata into R with mpactr::import_data() to create the mpactr R6 object. Apply the independent filters as a project-specific workflow: filter_mispicked_ions() for isotopic patterns incorrectly split during preprocessing, filter_group() for solvent-blank carryover or background components exceeding a user-defined abundance threshold, filter_cv() for ions inconsistent between technical replicates (CV-based), and filter_insource_ions() with cluster_threshold = 0.95 and copy_object = TRUE, which deconvolves via retention-time correlation matrices within MS1 scans and retains the highest-mass likely precursor ion. Summarize each filter with mpactr::filter_summary(..., filter_name), e.g. 'insource', listing failed (removed) and passed (retained) ions, and verify that failed + passed counts equal the pre-filter input ion count. Extract the filtered feature table with get_peak_table(), optionally group means with get_group_averages() and CVs with get_cv_data(), then save the filtered table and summaries as CSV.

## Related tools

- **mpactr** (Core package providing import_data(), the independent filters (filter_mispicked_ions, filter_group, filter_cv, filter_insource_ions), filter_summary(), and extractors get_peak_table(), get_metadata(), get_group_averages(), get_cv_data()) — https://github.com/mums2/mpactr
- **R** (Implementation language and runtime for the mpactr QC pipeline; installable from CRAN or GitHub)
- **ggplot** (Used with plotly to create interactive plots of input features and the filters they failed)
- **plotly** (Provides interactivity for the ggplot filter-failure plots)

## Examples

```
mp <- mpactr::import_data(peak_table, sample_meta); mp <- mpactr::filter_insource_ions(mp, cluster_threshold = 0.95, copy_object = TRUE); mpactr::filter_summary(mp, 'insource')
```

## Evaluation signals

- Accounting invariant: for each filter summary, failed ion count + passed ion count equals the pre-filter input ion count
- The extracted get_peak_table() output is a strict subset (rows) of the imported Progenesis peak table's features, with metadata columns intact
- In-source deconvolution with cluster_threshold = 0.95 removes the lower-mass fragment ions from each highly correlated retention-time group and retains the highest-mass ion
- Reproducibility check: re-running the documented filter sequence on the same PTY087I2 input reproduces identical failed/passed ion lists
- Saved CSV outputs parse cleanly and match the in-memory filter_summary and peak-table row counts

## Limitations

- All mpactr filters are independent; there is no mandated filter order, so a project-specific workflow must be justified by the user
- In-source fragment removal depends on retention-time correlation within MS1 scans, so the cluster_threshold choice (e.g., 0.95) directly governs which ions are removed
- The package addresses pre-processing artifacts (mispicked peaks, blank carryover, replicate variability, in-source fragments) but not instrument-level recalibration or MS2 spectral quality

## Evidence

- [readme] filter_insource_ions(): removal of fragment ions created during the first ionization in the tandem MS/MS workflow.: [evidence span withheld — anchor status: not_found]
- [abstract] mpactR automates pre-processing of LC-MS/MS data from microbiological samples: "mpactR automates pre-processing of liquid chromatography-tandem mass spectrometry (LC-MS/MS) data from microbiological samples"
- [methods] If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`: "If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`"
- [readme] All filters are independent, meaning they can be used to create a project-specific workflow: [evidence span withheld — anchor status: external_doc]
