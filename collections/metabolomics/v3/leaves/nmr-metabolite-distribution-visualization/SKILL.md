---
name: nmr-metabolite-distribution-visualization
description: 'Use when a CSV/TSV of Nightingale 1H-NMR metabolite concentrations (with
  optional phenotype columns) needs initial inspection: distribution checks, missingness
  overview, and comparison against the BBMRI-nl reference.'
when_to_use_negative:
- The goal is projecting or calibrating pre-trained metabolic scores (MetaboAge, surrogate
  clinical variables, COVID-severity, T2D) rather than exploratory distribution inspection
  — use the score-projection workflows instead
- Data are not Nightingale Health 1H-NMR metabolomics measurements and cannot be mapped
  to the required Nightingale metabolite identifiers via find_BBMRI_names
- A rigorous statistical comparison test between cohorts is required — MiMIR's data
  exploration provides visual/reference-overlay comparisons, not formal hypothesis
  testing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MiMIR
  role: 'R-shiny GUI hosting the data-exploration module: histograms, BBMRI reference
    overlays, NA heatmap, and correlation plots'
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R-shiny
  role: Interactive web framework through which the exploration module (file upload,
    plots, downloads) is delivered
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: ggplot2
  role: Rendering per-metabolite histograms and reference-overlay figures
- name: plotly
  role: Interactive rendering of correlation and distribution figures (plot_corply)
- name: heatmaply
  role: Interactive heatmaps for missingness overview and metabolite-by-phenotype
    correlation matrices
- name: matrixStats
  role: Computing per-metabolite summary statistics (mean, SD, quantiles) on the quantile-scaled
    matrix
- name: DT
  role: Interactive tables for the uploaded dataset summary and distribution statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:23174d40aa5b638b4d971a6cd88f03efd3b3bf6a8a6500700c01c73bf5f29c37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - DT
  - MiMIR
  - R
  - R-shiny
  - ggplot2
  - heatmaply
  - matrixStats
  - plotly
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# nmr-metabolite-distribution-visualization

## Summary

Visual exploration of newly uploaded Nightingale Health 1H-NMR metabolomics data within the MiMIR R-shiny app, comparing per-metabolite distributions of the uploaded cohort against the shipped BBMRI-nl reference histograms (scaled and unscaled) and rendering metabolite-by-phenotype correlation heatmaps.

## When to use

Use when a CSV/TSV of Nightingale 1H-NMR metabolite concentrations (with optional phenotype columns) needs initial inspection: distribution checks, missingness overview, and comparison against the BBMRI-nl reference.

## When NOT to use

- The goal is projecting or calibrating pre-trained metabolic scores (MetaboAge, surrogate clinical variables, COVID-severity, T2D) rather than exploratory distribution inspection — use the score-projection workflows instead
- Data are not Nightingale Health 1H-NMR metabolomics measurements and cannot be mapped to the required Nightingale metabolite identifiers via find_BBMRI_names
- A rigorous statistical comparison test between cohorts is required — MiMIR's data exploration provides visual/reference-overlay comparisons, not formal hypothesis testing

## Inputs

- CSV or TSV file of Nightingale Health 1H-NMR metabolomics measurements with column names matching the synthetic_metabolic_dataset example
- Optional phenotypic variables file (synthetic_phenotypic_dataset-style naming)
- Shipped MiMIR reference objects: BBMRI_hist, BBMRI_hist_scaled, metabo_names_translator, and the BBMRI-nl 1H-NMR metabolomics repository histograms

## Outputs

- Per-metabolite histogram plots with overlaid BBMRI-nl reference curves (scaled and unscaled variants), exportable as PNG
- Per-metabolite distribution-summary table (mean, SD, quantiles) written to CSV
- Missingness heatmap (plot_na_heatmap) and dataset dimension report
- Metabolite-by-phenotype correlation heatmap (plot_corply with resort_on_p / abs options)

## How to apply

Load the metabolomics CSV/TSV (and optional phenotype file) into MiMIR, mapping Nightingale metabolite identifiers with find_BBMRI_names and the metabo_names_translator dataset so column names match the synthetic_metabolic_dataset example. First summarize table dimensions and missingness (report.dim, plot_na_heatmap), flagging metabolites/samples against the Nmax_zero and Nmax_miss thresholds used by QCprep, and impute or drop via impute_miss/subset_samples_miss. Compute per-metabolite summaries (mean, SD, quantiles) with matrixStats on quantile-scaled data (apply.scale), then render per-metabolite histograms with hist_plots/multi_hist and overlay the shipped BBMRI_hist and BBMRI_hist_scaled reference histograms via BBMRI_hist_plot to reveal cohort shifts (e.g. serum_c and apoa1 elevated in LLS_PAROFFs vs BBMRI-nl). Finally compute the metabolite-by-phenotype correlation matrix with cor_assoc and render it with plot_corply using resort_on_p, abs, and correlation-absolute options; export the reference-comparison figure as PNG and the summary table as CSV.

## Related tools

- **MiMIR** (R-shiny GUI hosting the data-exploration module: histograms, BBMRI reference overlays, NA heatmap, and correlation plots) — https://github.com/DanieleBizzarri/MiMIR
- **R-shiny** (Interactive web framework through which the exploration module (file upload, plots, downloads) is delivered) — https://github.com/DanieleBizzarri/MiMIR
- **ggplot2** (Rendering per-metabolite histograms and reference-overlay figures)
- **plotly** (Interactive rendering of correlation and distribution figures (plot_corply))
- **heatmaply** (Interactive heatmaps for missingness overview and metabolite-by-phenotype correlation matrices)
- **matrixStats** (Computing per-metabolite summary statistics (mean, SD, quantiles) on the quantile-scaled matrix)
- **DT** (Interactive tables for the uploaded dataset summary and distribution statistics)

## Examples

```
library(MiMIR); MiMIR::startApp()
```

## Evaluation signals

- All uploaded Nightingale metabolite columns are recognized (find_BBMRI_names / metabo_names_translator resolve names to the shipped example-dataset identifiers)
- Histogram overlays show the uploaded cohort's density curves plotted against both BBMRI_hist (unscaled) and BBMRI_hist_scaled references for each metabolite
- Per-metabolite summary CSV contains mean, SD, and quantile columns for every retained metabolite after QC flags (Nmax_zero / Nmax_miss) are applied
- Correlation heatmap cells correspond to cor_assoc output and re-sort correctly under resort_on_p and absolute-correlation options
- Exported figures are valid PNGs and the summary table round-trips as CSV with the expected number of metabolite rows

## Limitations

- Column naming is strict: uploads must use the same column names as the synthetic example dataset (CSV/TSV accepted); unmapped metabolite identifiers fail recognition
- QC thresholds Nmax_zero and Nmax_miss can drop metabolites/samples, so distributions are only shown for features surviving QCprep filtering
- Comparison against BBMRI_hist/BBMRI_hist_scaled is a visual overlay against the BBMRI-nl reference cohort, not a calibrated statistical test; app issues may require manually installing the listed shiny/statistics/imaging dependencies

## Evidence

- [intro] Explore new Nightingale measurements: "It allows to easily explore new metabolomics measurements assayed by Nightingale Health"
- [readme] GUI for ad-hoc statistical analysis of 1H-NMR: "graphical user interface that provides an intuitive framework for ad-hoc statistical analysis of 1H-NMR metabolomics by Nightingale Health"
- [methods] Upload format requirement: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
- [readme] Example datasets define naming: "These example dataset can be used to test the App and to understand how the variables in your own dataset should be named."
- [methods] QC filter functions: "`QCprep(mat, PARAM_metaboAge, quiet, Nmax_zero, Nmax_miss)`"
