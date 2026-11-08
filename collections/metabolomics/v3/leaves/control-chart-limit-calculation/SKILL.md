---
name: control-chart-limit-calculation
description: Use when pooled QC samples are injected at regular intervals in a sequence
  and you need to flag 'out-of-control' observations or signal drifts per chemical
  descriptor before releasing a metabolomics/lipidomics batch.
when_to_use_negative:
- The input is already a feature table of study samples without repeated pooled-QC
  injections, so no control limits can be estimated from QCpool data.
- The goal is to handle missing values or truly absent data rather than to detect
  out-of-control observations — those are separate QComics steps.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: Provides the Shiny app and helper functions (make_qc_graph, make_nice_table)
    that compute QCpool means/SDs, draw run-sequence control charts with ±SD/±2SD/±3SD
    limits, and apply Westgard rules to flag out-of-control observations
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics package is installed and run (RStudio
    or shiny server)
  repo: github.com/ricoderks/QComics
- name: ggplot2
  role: Plotting backend used by make_qc_graph to render the run-sequence control
    charts with SD-based limit lines
  repo: github.com/ricoderks/QComics
- name: Sciex Multiquant
  role: Source software that quantifies QCpool injections and exports the chemical-descriptor
    table consumed by the control-chart workflow (must be > v3.0.3)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:a784f720d34eadc277b8e595e2f0f32e96457a76a8f8422f5eff746804707709
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - Sciex Multiquant
  - dplyr
  - ggplot2
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# control-chart-limit-calculation

## Summary

Builds run-sequence Shewhart control charts for each chemical descriptor of pooled QC (QCpool) injections, setting limits at ±1SD, ±2SD, and ±3SD around the QCpool mean and applying the four Westgard rules to decide whether a batch is in control.

## When to use

Use when pooled QC samples are injected at regular intervals in a sequence and you need to flag 'out-of-control' observations or signal drifts per chemical descriptor before releasing a metabolomics/lipidomics batch.

## When NOT to use

- The input is already a feature table of study samples without repeated pooled-QC injections, so no control limits can be estimated from QCpool data.
- The goal is to handle missing values or truly absent data rather than to detect out-of-control observations — those are separate QComics steps.

## Inputs

- QCpool chemical-descriptor table (data_df) exported from Sciex Multiquant (> v3.0.3) as txt
- Injection sequence/order for each QCpool sample

## Outputs

- Per-descriptor run-sequence control-chart figures (one file per chemical descriptor)
- westgard_violations.csv listing injection-descriptor pairs, the triggering Westgard rule, and the sample descriptors exceeding ±SD, ±2SD, or ±3SD limits

## How to apply

Load the QCpool chemical-descriptor series (data_df) with its injection sequence in R (via the QComics Shiny app or standalone scripts) and compute the mean and SD of every descriptor across the QCpool injections. For each descriptor, build a run-sequence control chart (make_qc_graph(data_df, yaxis, sequence) with ggplot2) drawing control limits at ±1SD, ±2SD, and ±3SD around the QCpool mean, mirroring the upper and lower limits at ±SD, ±2SD, and ±3SD described in the article. Evaluate the four Westgard rules on each descriptor series: a single point beyond ±3SD; two consecutive points beyond ±2SD on the same side of the mean; four consecutive points beyond ±1SD on the same side; and ten consecutive points on the same side of the mean. Record every injection-descriptor pair that violates any rule, noting which rule triggered, and render one control-chart figure per descriptor while exporting the flagged violations as westgard_violations.csv (tabulated by sequence as in make_nice_table(data_df, sequence)). When the criteria are not met, the neighboring study samples are scrutinized for possible discard and reanalysis, consistent with QComics' step of detecting 'out-of-control' observations.

## Related tools

- **QComics** (Provides the Shiny app and helper functions (make_qc_graph, make_nice_table) that compute QCpool means/SDs, draw run-sequence control charts with ±SD/±2SD/±3SD limits, and apply Westgard rules to flag out-of-control observations) — github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics package is installed and run (RStudio or shiny server)) — github.com/ricoderks/QComics
- **ggplot2** (Plotting backend used by make_qc_graph to render the run-sequence control charts with SD-based limit lines) — github.com/ricoderks/QComics
- **Sciex Multiquant** (Source software that quantifies QCpool injections and exports the chemical-descriptor table consumed by the control-chart workflow (must be > v3.0.3))

## Examples

```
QComics::run_app()
```

## Evaluation signals

- The R script runs without error and produces one control-chart figure file per chemical descriptor plus westgard_violations.csv.
- The emitted violation flags agree with an expert-reviewed reference set of out-of-control QCpool injections.
- Control-limit lines on every chart are drawn at exactly ±1SD, ±2SD, and ±3SD around the QCpool mean for that descriptor.
- All four Westgard rules (1×3SD, 2×2SD same-side, 4×1SD same-side, 10×same-side) are evaluated, and each violation record identifies the triggering rule.
- Violations list is keyed by injection sequence and descriptor, matching the tabulated order produced by make_nice_table(data_df, sequence).

## Limitations

- QCpool samples must be injected at regular intervals during one or more sequences; without repeated pooled-QC measurements no control limits can be computed.
- The exported QCpool data must come from Sciex Multiquant (> v3.0.3) in txt format — other vendor formats are not covered by the documented workflow.
- QComics is marked 'experimental' in its lifecycle badge, so the control-chart implementation may still change.
- When a Westgard criterion is not met, neighboring study samples must be manually scrutinized for possible discard and reanalysis; the skill does not automatically remove study samples.

## Evidence

- [other] QComics flags 'out-of-control' QC samples by plotting each chemical descriptor over run order with upper and lower limits at ±SD, ±2SD, and ±3SD, then applying Westgard control rules: [evidence span withheld — anchor status: not_found]
- [methods] 1. Load the QCpool chemical-descriptor series (data_df) with its injection sequence in R and compute the mean and SD of every descriptor across the QCpool injections.: [evidence span withheld — anchor status: not_found]
- [readme] The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study.: "The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study. For this, a pooled sample (QCpool) needs to measured in regular intervals during"
- [readme] These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format.: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
- [abstract] (ii) detect signal drifts and “out-of-control” observations: "(ii) detect signal drifts and “out-of-control” observations"
