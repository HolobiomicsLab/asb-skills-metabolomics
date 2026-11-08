---
name: metabolomics-qc-outlier-flagging
description: Use when a pooled-QC-injected targeted metabolomics/lipidomics sequence
  (Sciex MultiQuant txt exports) needs outlying samples flagged before statistical
  analysis, and you need defensible, reportable outlier criteria based on PCA scores.
when_to_use_negative:
- 'Data is unprocessed raw signal: QComics outlier removal is step (iii) of the sequential
  workflow, which should first correct for background noise and carryover and detect
  signal drifts (steps i–ii)'
- Data contains missing values and truly absent features that have not yet been separated,
  since QComics requires separately handling missing values and truly absent data
  to avoid losing relevant biological information
- The dataset lacks regularly injected pooled QC (QCpool) samples, which the QComics
  package requires as its monitoring basis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: R/Shiny package implementing the sequential multistep QC protocol; provides
    PCA-based Hotelling T2 and DmodX outlier flagging plus make_qc_graph and make_nice_table
    plotting/reporting helpers
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics Shiny app is run for outlier detection
    and flagging
  repo: https://github.com/ricoderks/QComics
- name: RStudio
  role: IDE used to run the app locally via QComics::run_app()
- name: ggplot2
  role: Underlying plotting used to render the T2 range and DmodX figures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:534ff31dbb82ad5272b395f067fe739abb862237f7ab800abd81f86b81979a1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - RStudio
  - ggplot2
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metabolomics-qc-outlier-flagging

## Summary

Flags outlying samples in metabolomics QC data by fitting a PCA model to the processed QC data and identifying samples outside the 95% Hotelling T2 confidence ellipse, complemented by DmodX residual-distance screening, as step (iii) 'removal of outlying samples' of the QComics sequential QC workflow.

## When to use

Use when a pooled-QC-injected targeted metabolomics/lipidomics sequence (Sciex MultiQuant txt exports) needs outlying samples flagged before statistical analysis, and you need defensible, reportable outlier criteria based on PCA scores.

## When NOT to use

- Data is unprocessed raw signal: QComics outlier removal is step (iii) of the sequential workflow, which should first correct for background noise and carryover and detect signal drifts (steps i–ii)
- Data contains missing values and truly absent features that have not yet been separated, since QComics requires separately handling missing values and truly absent data to avoid losing relevant biological information
- The dataset lacks regularly injected pooled QC (QCpool) samples, which the QComics package requires as its monitoring basis

## Inputs

- Processed metabolomics QC data table (data_df) with pooled QCpool samples measured at regular intervals across one or more sequences
- Sciex MultiQuant (> v3.0.3) exported txt files from the targeted assay sequence

## Outputs

- Hotelling T2 range plot with 95% confidence ellipse flagging out-of-control samples
- DmodX residual-distance plot flagging samples with variance unexplained by the PCA model
- outlier_samples.csv listing the union of flagged outlier samples (formatted via make_nice_table)

## How to apply

Install the QComics R package with remotes::install_github('ricoderks/QComics') and load the processed metabolomics QC data table (data_df) into R/RStudio, launching the Shiny app via QComics::run_app() (or QComics::run_in_browser()). Fit a PCA model to the processed data and compute per-sample Hotelling T2 distances and DmodX residual distances; render the T2 range plot with make_qc_graph(data_df, yaxis, sequence) and flag samples falling outside the 95% Hotelling T2 confidence ellipse (extreme T2 values). Render the DmodX plot with make_qc_graph and complementarily flag samples whose variance is unexplained (high residual distance) relative to the PCA model. Export the union of flagged samples as outlier_samples.csv, formatted with make_nice_table(data_df, sequence). Per QComics guidance, exclude a flagged sample only when raw-data anomalies (e.g., improper collection, preprocessing, or storage identified via quality markers) explain its behavior; QComics criteria are adapted from bioanalytical method validation guidelines to keep QC harmonized across laboratories.

## Related tools

- **QComics** (R/Shiny package implementing the sequential multistep QC protocol; provides PCA-based Hotelling T2 and DmodX outlier flagging plus make_qc_graph and make_nice_table plotting/reporting helpers) — https://github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics Shiny app is run for outlier detection and flagging) — https://github.com/ricoderks/QComics
- **RStudio** (IDE used to run the app locally via QComics::run_app())
- **ggplot2** (Underlying plotting used to render the T2 range and DmodX figures)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- R script / Shiny app runs without error after remotes::install_github('ricoderks/QComics')
- Both the Hotelling T2 range figure and the DmodX figure files exist and render per-sample distances along the injection sequence
- The flagged sample set matches the stated criteria: samples outside the 95% T2 ellipse, unioned with samples showing high DmodX residual variance
- Flagged outliers in outlier_samples.csv are confirmed consistent with the ellipse/residual criteria by expert_review
- Exclusions are restricted to samples whose raw-data anomalies (quality-marker evidence) explain the outlying behavior

## Limitations

- QComics is labeled experimental (lifecycle badge) and targets pooled-QC-injected targeted metabolomics/lipidomics data exported from Sciex MultiQuant txt files
- PCA/Hotelling T2 and DmodX flagging alone is not sufficient for exclusion; the protocol recommends exclusion only when raw-data anomalies explain the behavior, so unexplained outliers should be investigated rather than automatically dropped
- The paper provides no quantitative benchmark or validation metrics for the outlier step, and existing metabolomics QC practices lack predefined quality criteria, making cross-laboratory harmonization dependent on the adapted bioanalytical validation guidelines

## Evidence

- [abstract] PCA-based T2 ellipse criterion: [evidence span withheld — anchor status: not_found]
- [intro] outlier removal as workflow step: [evidence span withheld — anchor status: not_found]
- [intro] criteria adapted from validation guidelines: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [readme] installation command: "remotes::install_github("ricoderks/QComics")"
- [readme] app launch requirement: "The goal of the `QComics` package is to have a quick overview of the
quality of a metabolomics or lipidomics study."
