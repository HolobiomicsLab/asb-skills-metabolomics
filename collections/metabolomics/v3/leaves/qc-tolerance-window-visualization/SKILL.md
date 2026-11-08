---
name: qc-tolerance-window-visualization
description: Use when repeated QCpool injections from a metabolomics/lipidomics sequence
  must be plotted over the injection sequence with tolerance-window bands per descriptor
  (RSD- and ppm-based) to detect drift and report overall precision/accuracy.
when_to_use_negative:
- The data are not from repeated pooled-QC (QCpool) injections analyzed with Sciex
  Multiquant, e.g., single-shot samples without a QC sequence
- Only NMR or non-MS omics data lacking intensity/RT/peak-width/m/z descriptors are
  available, since the tolerance criteria are MS-descriptor-specific
- A rigorous statistical drift-correction method is required rather than visual monitoring
  — QComics is positioned as an easily implementable, reportable monitoring workflow,
  not advanced statistical modeling
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: QComics
  role: Shiny app that renders the time-series QC plots with tolerance windows for
    the pooled-QC descriptors
  repo: github.com/ricoderks/QComics
- name: R / RStudio
  role: Runtime environment from which the QComics app is launched for plotting
- name: ggplot2
  role: Graphics backend for the per-descriptor time-series QC graphs with tolerance-window
    bands
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:bf1e23a47ba2b30c3b6871ed513e76a213f70d84bf410ee3871c05ef6f69d238
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DT
  - QComics
  - R
  - R / RStudio
  - RStudio
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# qc-tolerance-window-visualization

## Summary

Generate time-series QC plots per descriptor (peak intensity, retention time, peak width, m/z error) from repeated pooled-QC injections, overlaying predefined tolerance-window bands so signal drifts and out-of-control observations in metabolomics/lipidomics sequences are visually reportable.

## When to use

Use when repeated QCpool injections from a metabolomics/lipidomics sequence must be plotted over the injection sequence with tolerance-window bands per descriptor (RSD- and ppm-based) to detect drift and report overall precision/accuracy.

## When NOT to use

- The data are not from repeated pooled-QC (QCpool) injections analyzed with Sciex Multiquant, e.g., single-shot samples without a QC sequence
- Only NMR or non-MS omics data lacking intensity/RT/peak-width/m/z descriptors are available, since the tolerance criteria are MS-descriptor-specific
- A rigorous statistical drift-correction method is required rather than visual monitoring — QComics is positioned as an easily implementable, reportable monitoring workflow, not advanced statistical modeling

## Inputs

- QCpool feature table (data_df) exported from Sciex Multiquan (> v3.0.3) in txt format
- Injection sequence/order metadata for the QCpool samples
- Descriptor measurements: peak intensity, retention time, peak width, and m/z (HRMS)

## Outputs

- Time-series QC plots per descriptor with predefined tolerance-window bands (ggplot2/make_qc_graph)
- Per-descriptor pass/fail QC assessment table (e.g., qc_descriptor_assessment.csv rendered with make_nice_table/DT)

## How to apply

Launch the QComics Shiny app from R/RStudio via QComics::run_app() (or QComics::run_in_browser()) and load the QCpool feature table exported from Sciex Multiquant as txt, together with its injection sequence. For each descriptor compute precision across the repeated QCpool injections (RSD of peak intensity, retention time, and peak width) and, for HRMS, the m/z error in ppm, then apply QComics' acceptance criteria adapted from bioanalytical method validation guidelines: RSD < 30% for peak intensity, RSD < 2% for RT, RSD < 15% for peak width, and m/z error < 10 ppm. Render each descriptor as a time series over the run order using ggplot2 (make_qc_graph), shading a tolerance-window band so points outside the band flag drift or out-of-control injections; the final assessment is reportable as figures (e.g., Figures 2–4) and tabulated RSD estimations. Judgment of correct application: each plot shows the QCpool trend across the sequence, the tolerance band matches the stated criterion, and out-of-band points are identifiable per descriptor.

## Related tools

- **QComics** (Shiny app that renders the time-series QC plots with tolerance windows for the pooled-QC descriptors) — github.com/ricoderks/QComics
- **R / RStudio** (Runtime environment from which the QComics app is launched for plotting)
- **ggplot2** (Graphics backend for the per-descriptor time-series QC graphs with tolerance-window bands)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- Each descriptor plot spans the full injection sequence and shows QCpool measurements in run order, not pooled statistics
- Tolerance bands match the stated criteria: ±30% RSD for intensity, ±2% for RT, ±15% for peak width, <10 ppm m/z error (HRMS only)
- Points falling outside the tolerance window are visually distinguishable, corresponding to 'out-of-control' observations
- The rendered assessment includes tabulated RSD estimations alongside the figures for reportability
- Plots reproduce from the documented example QCpool feature table when the app is launched per README instructions

## Limitations

- QComics is lifecycle:experimental (per README badge), so plotting behavior may change between versions
- Requires QCpool data analyzed and exported from Sciex Multiquant (> v3.0.3) as txt; other vendor formats are not described
- Quality criteria were adapted from bioanalytical method validation guidelines and may need adjustment for heterogeneous data sets with xenobiotics present at very different concentrations
- The tool can be extended beyond MS-based metabolomics, but the descriptor criteria and plots were designed for MS data

## Evidence

- [abstract] robust, easily implementable and reportable method for monitoring and controlling data quality: "we have developed QComics, a robust, easily implementable and reportable method for monitoring and controlling data quality"
- [intro] The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [readme] a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences: "For this, a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences. These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported"
- [methods] From Rstudio run the app as: [evidence span withheld — anchor status: not_found]
