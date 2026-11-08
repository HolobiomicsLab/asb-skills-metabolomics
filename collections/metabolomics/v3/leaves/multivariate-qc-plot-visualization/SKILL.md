---
name: multivariate-qc-plot-visualization
description: Use when a metabolomics/lipidomics sequence contains regularly injected
  QCpool samples and you must identify outlying or out-of-control samples that univariate
  plots miss, before outlier removal in the QComics sequential workflow.
when_to_use_negative:
- The study has no QCpool samples measured at regular intervals during the sequence(s)
  — the package requires pooled samples injected regularly to monitor quality
- The QC data were not exported from Sciex MultiQuant (> v3.0.3) to txt format, so
  the QComics app cannot parse the input
- The goal is correction of background noise/carryover, missing-data handling, or
  quality-marker monitoring — those are earlier/later steps in the QComics sequential
  workflow, not multivariate outlier visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: Shiny-based R package that fits the PCA model, computes Hotelling T2 and DmodX
    distances, and renders the QC graphs (make_qc_graph, make_nice_table)
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment for installing QComics and executing the QC visualization
    workflow
- name: RStudio
  role: IDE from which the QComics app is launched via QComics::run_app()
- name: ggplot2
  role: Plotting backend used to render the T2 and DmodX QC figures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:260363ebb4d1f76d88dd6e09b7671cb1ee2e50bb161ed777209cd5493d7f3c73
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# multivariate-qc-plot-visualization

## Summary

Multivariate quality-control visualization from the QComics protocol: fit a PCA model to processed metabolomics QC data and render Hotelling T2 (95% confidence ellipse) and DmodX residual plots via make_qc_graph to flag 'out-of-control' samples.

## When to use

Use when a metabolomics/lipidomics sequence contains regularly injected QCpool samples and you must identify outlying or out-of-control samples that univariate plots miss, before outlier removal in the QComics sequential workflow.

## When NOT to use

- The study has no QCpool samples measured at regular intervals during the sequence(s) — the package requires pooled samples injected regularly to monitor quality
- The QC data were not exported from Sciex MultiQuant (> v3.0.3) to txt format, so the QComics app cannot parse the input
- The goal is correction of background noise/carryover, missing-data handling, or quality-marker monitoring — those are earlier/later steps in the QComics sequential workflow, not multivariate outlier visualization

## Inputs

- Processed metabolomics QC data table (data_df) with per-sample measurements across the sequence
- QCpool samples exported from Sciex MultiQuant (> v3.0.3) in txt format
- Sequence/run metadata (sample sequence, y-axis variable selection) for make_qc_graph

## Outputs

- Hotelling T2 range plot showing samples relative to the 95% confidence ellipse
- DmodX residual-distance plot flagging high unexplained variance
- outlier_samples.csv listing the union of T2-ellipse and DmodX-flagged samples

## How to apply

Load the processed metabolomics QC data table (data_df) into R after installing QComics from github.com/ricoderks/QComics with remotes::install_github(). Fit a PCA model to the processed data and compute per-sample Hotelling T2 distances and DmodX residual distances. Render the T2 range plot with make_qc_graph(data_df, yaxis, sequence) and flag samples falling outside the 95% Hotelling T2 confidence ellipse; then render the DmodX plot with the same call and complementarily flag samples whose variance is unexplained by the PCA model (high residual distance). Export the union of flagged samples as outlier_samples.csv, formatted with make_nice_table(data_df, sequence), and exclude samples only when raw-data anomalies (e.g., injection or acquisition problems) explain the outlying behavior.

## Related tools

- **QComics** (Shiny-based R package that fits the PCA model, computes Hotelling T2 and DmodX distances, and renders the QC graphs (make_qc_graph, make_nice_table)) — https://github.com/ricoderks/QComics
- **R** (Runtime environment for installing QComics and executing the QC visualization workflow)
- **RStudio** (IDE from which the QComics app is launched via QComics::run_app())
- **ggplot2** (Plotting backend used to render the T2 and DmodX QC figures)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The R script executes without error end-to-end (install, run_app, graph generation)
- Both figure files exist: the T2 range plot and the DmodX plot for the sequence
- Flagged samples are consistent with the stated criteria: outside the 95% Hotelling T2 ellipse or extreme T2 values, plus high DmodX residual variance — confirmed by expert_review
- outlier_samples.csv exists, contains the union (not the intersection) of T2- and DmodX-flagged samples, and is formatted via make_nice_table
- Exclusion decisions are traceable to raw-data anomalies rather than being made on plot appearance alone

## Limitations

- QComics has an experimental lifecycle badge, and no quantitative benchmark or validation metrics are reported in the article for the outlier-detection step
- The 95% Hotelling T2 ellipse and DmodX criteria are heuristic; the protocol recommends exclusion only when raw-data anomalies explain the behavior, so flagged samples should not be removed automatically
- Requires pooled QC samples injected at regular intervals and Sciex MultiQuant txt export; studies without regular QCpool injections cannot be visualized with this workflow

## Evidence

- [abstract] Abstract — sequential QC steps including out-of-control detection: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations,"
- [discussion] Discussion — outlier detection/removal as a protocol step: "detection and removal of outliers"
- [discussion] Discussion — QComics as a sequential multistep QC protocol: "we have developed a sequential multistep QC protocol, termed as QComics"
- [readme] README — package purpose for metabolomics/lipidomics QC overview: "The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study."
- [readme] README — requirement of regular QCpool injections: "For this, a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences."
