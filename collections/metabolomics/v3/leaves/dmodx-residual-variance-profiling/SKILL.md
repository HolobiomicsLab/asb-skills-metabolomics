---
name: dmodx-residual-variance-profiling
description: Use when processed pooled-QC (QCpool) metabolomics or lipidomics data
  must be screened for outliers whose behavior is not explained by the main PCA structure,
  i.e., samples with high residual distance-to-model.
when_to_use_negative:
- Data are raw instrument files rather than a processed metabolomics QC table — background
  noise, carryover, and signal drift should be corrected first in the QComics sequential
  workflow
- The goal is assessing missing values versus truly absent data or quality markers
  for preanalytical errors — these are separate QComics workflow steps, not outlier
  profiling
- The study lacks regularly measured pooled QC samples (QCpool), which QComics requires
  as input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QComics
  role: R/Shiny package providing PCA-based outlier detection (Hotelling T2 95% ellipse
    plus DmodX) within its sequential multistep QC workflow; installed from GitHub
    and run via run_app()
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics app is launched and PCA/T2/DmodX
    statistics are computed
- name: RStudio
  role: Used to run the app locally via QComics::run_app()
- name: ggplot2
  role: Graphics backend for rendering the T2 range and DmodX QC plots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:a4fc5d4d0f324225359f6d50bde95236c7dcc07276fbd07443ce9e2c09f6d9e9
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dmodx-residual-variance-profiling

## Summary

Flags outlying samples in metabolomics QC data by profiling DmodX residual variance from a fitted PCA model, complementing Hotelling T2 distance screening in the QComics outlier-removal step.

## When to use

Use when processed pooled-QC (QCpool) metabolomics or lipidomics data must be screened for outliers whose behavior is not explained by the main PCA structure, i.e., samples with high residual distance-to-model.

## When NOT to use

- Data are raw instrument files rather than a processed metabolomics QC table — background noise, carryover, and signal drift should be corrected first in the QComics sequential workflow
- The goal is assessing missing values versus truly absent data or quality markers for preanalytical errors — these are separate QComics workflow steps, not outlier profiling
- The study lacks regularly measured pooled QC samples (QCpool), which QComics requires as input

## Inputs

- Processed metabolomics QC data table (data_df) with pooled QCpool samples measured at regular intervals in one or more sequences
- Sciex Multiquant (> v3.0.3) exported txt files of QCpool analyses
- Fitted PCA model with per-sample Hotelling T2 and DmodX residual distances

## Outputs

- DmodX residual distance plot rendered via make_qc_graph(data_df, yaxis, sequence)
- Union of flagged outlier samples exported as outlier_samples.csv
- Formatted sample listing produced with make_nice_table(data_df, sequence)

## How to apply

After installing QComics with remotes::install_github('ricoderks/QComics'), load the processed metabolomics QC data table (data_df) into R and fit a PCA model to the processed data, computing per-sample Hotelling T2 distances and DmodX residual distances. Render the DmodX plot with make_qc_graph(data_df, yaxis, sequence) and flag samples showing high residual variance relative to the PCA model — i.e., samples whose variation is unexplained by the PCA model, complementing those outside the 95% Hotelling T2 confidence ellipse. Combine the DmodX flags with T2 flags as the union of candidate outliers, but recommend exclusion only when raw-data anomalies explain the flagged behavior, consistent with the QComics step for removal of outlying samples. Export the union of flagged samples via make_nice_table(data_df, sequence) as outlier_samples.csv. Rationale: DmodX catches samples whose variance is unexplained by the retained PCA components, which T2 alone can miss.

## Related tools

- **QComics** (R/Shiny package providing PCA-based outlier detection (Hotelling T2 95% ellipse plus DmodX) within its sequential multistep QC workflow; installed from GitHub and run via run_app()) — https://github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics app is launched and PCA/T2/DmodX statistics are computed)
- **RStudio** (Used to run the app locally via QComics::run_app())
- **ggplot2** (Graphics backend for rendering the T2 range and DmodX QC plots)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The R script runs without error after remotes::install_github('ricoderks/QComics')
- The DmodX figure file (and T2 range plot) exist on disk after rendering via make_qc_graph
- The flagged sample set is consistent with the ellipse/residual criteria as confirmed by expert_review
- outlier_samples.csv contains exactly the union of samples outside the 95% Hotelling T2 ellipse and samples with extreme DmodX residual variance
- Exclusions are restricted to samples whose behavior is explained by raw-data anomalies

## Limitations

- QComics is experimental (lifecycle badge) and its README requires QCpool samples analyzed with Sciex Multiquant (> v3.0.3) and exported to txt format
- DmodX flags must be complemented by evidence: exclusion is recommended only when raw-data anomalies explain the behavior, since QComics quality criteria are adapted from bioanalytical validation guidelines rather than universal cutoffs
- No quantitative benchmark or validation metrics are reported for the protocol, and Figures 2–4 illustrating the steps are not present in the provided text

## Evidence

- [other] QComics identifies outliers via PCA with Hotelling T2 statistics (samples outside the 95% ellipse or extreme T2 values) and complements this with DmodX to flag samples whose variance is unexplained by the PCA model: [evidence span withheld — anchor status: not_found]
- [abstract] The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and out-of-control observations, (iii) deal with missing data,: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts"
- [readme] remotes::install_github("ricoderks/QComics"): "You can install the development version of QComics like so:

``` r
remotes::install_github("ricoderks/QComics")
```"
- [readme] a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences. These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format.: "a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences"
