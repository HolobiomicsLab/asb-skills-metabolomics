---
name: pca-hotelling-t2-outlier-detection
description: Use when a metabolomics/lipidomics data table with QCpool samples measured
  at regular sequence intervals must be screened for outlying samples before downstream
  analysis, i.e. during the 'removal of outlying samples' step of a multistep QC protocol.
when_to_use_negative:
- Data have not yet passed the preceding QComics steps (background noise/carryover
  correction, signal-drift detection, missing-value handling) — outliers flagged at
  this point may reflect uncorrected drift rather than true sample anomalies
- A sample falls outside the T2 ellipse but no raw-data anomaly explains its behavior
  — the protocol recommends exclusion only when raw-data anomalies justify it
- The dataset is not MS-based metabolomics/lipidomics with pooled QC samples; QComics
  was designed for Sciex Multiquant-exported targeted metabolomics data, though it
  reportedly extends to other omics and MS data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QComics
  role: R/Shiny package implementing the PCA-based outlier detection step (Hotelling
    T2 ellipse + DmodX) of the sequential QC workflow
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics Shiny app and plotting functions
    are executed
- name: RStudio
  role: IDE from which the app is run locally via QComics::run_app()
- name: ggplot2
  role: Graphics backend for rendering the T2 and DmodX QC plots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:139b209bf2aa0f4cf4b2938b2f9d4d16b92a436f8ca82a500fc900279aa20ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - RStudio
  - ggplot2
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# pca-hotelling-t2-outlier-detection

## Summary

Detect outlying samples in processed metabolomics data by fitting a PCA model and flagging samples outside the 95% Hotelling T2 confidence ellipse, complemented by DmodX residual-distance screening — the outlier-removal step of the QComics sequential QC workflow.

## When to use

Use when a metabolomics/lipidomics data table with QCpool samples measured at regular sequence intervals must be screened for outlying samples before downstream analysis, i.e. during the 'removal of outlying samples' step of a multistep QC protocol.

## When NOT to use

- Data have not yet passed the preceding QComics steps (background noise/carryover correction, signal-drift detection, missing-value handling) — outliers flagged at this point may reflect uncorrected drift rather than true sample anomalies
- A sample falls outside the T2 ellipse but no raw-data anomaly explains its behavior — the protocol recommends exclusion only when raw-data anomalies justify it
- The dataset is not MS-based metabolomics/lipidomics with pooled QC samples; QComics was designed for Sciex Multiquant-exported targeted metabolomics data, though it reportedly extends to other omics and MS data

## Inputs

- Processed metabolomics data table (data_df) with QCpool samples measured at regular intervals in one or more sequences
- QCpool sample annotations exported from Sciex Multiquant (> v3.0.3) as txt files
- Sequence/run order metadata for per-sample plotting

## Outputs

- Hotelling T2 range plot with 95% confidence ellipse highlighting out-of-control samples
- DmodX plot flagging samples with high residual variance relative to the PCA model
- outlier_samples.csv listing the union of flagged outlier samples (formatted via make_nice_table)

## How to apply

Load the processed metabolomics data table (data_df) into R after installing the QComics package with remotes::install_github("ricoderks/QComics") and launching the Shiny app via QComics::run_app(). Fit a PCA model to the processed data and compute per-sample Hotelling T2 distances and DmodX residual distances. Render the T2 range plot with make_qc_graph(data_df, yaxis, sequence) and flag samples falling outside the 95% Hotelling T2 confidence ellipse; then render the DmodX plot to complementarily flag samples whose variance is unexplained by the PCA model. Export the union of flagged samples as outlier_samples.csv, formatted with make_nice_table(data_df, sequence), and recommend exclusion only when raw-data anomalies explain the outlying behavior — the QComics protocol deliberately places this step after initial drift/contamination exploration and missing-value handling.

## Related tools

- **QComics** (R/Shiny package implementing the PCA-based outlier detection step (Hotelling T2 ellipse + DmodX) of the sequential QC workflow) — github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics Shiny app and plotting functions are executed)
- **RStudio** (IDE from which the app is run locally via QComics::run_app())
- **ggplot2** (Graphics backend for rendering the T2 and DmodX QC plots)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The R script runs without error end to end (QComics installed, app launched, plots generated)
- Both the Hotelling T2 figure and the DmodX figure files exist as rendered outputs
- The flagged sample set is consistent with the ellipse/residual criteria: samples outside the 95% T2 ellipse or with extreme T2 values, plus samples with high DmodX residual variance, are confirmed by expert review
- outlier_samples.csv contains exactly the union of T2-flagged and DmodX-flagged samples and is formatted by make_nice_table

## Limitations

- Exclusion is recommended only when raw-data anomalies explain the outlying behavior — statistical flagging alone is insufficient justification to remove a sample
- QComics is marked experimental (lifecycle badge) and expects QCpool samples analysed with Sciex Multiquant (> v3.0.3) and exported to txt format; other export formats are not covered by the README
- No quantitative benchmark or validation metrics are reported for the protocol; prior applications are cited (refs 43-50) but supporting material is not included

## Evidence

- [abstract] Outlier detection via T2 ellipse and DmodX: "detect signal drifts and “out-of-control” observations"
- [intro] Sequential step context: "(iii) removal of outlying samples"
- [discussion] Outlier detection is a named filter step: "detection and removal of outliers"
- [readme] Installation command: "remotes::install_github("ricoderks/QComics")"
- [readme] Input data requirement: "a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences. These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt`"
