---
name: run-sequence-quality-charting
description: Use when targeted metabolomics or lipidomics sequences contain regularly
  injected QCpool samples exported from Sciex MultiQuant (> v3.0.3) as txt files,
  and you need to detect drifts, contaminants, or out-of-control measurements before
  further QC steps.
when_to_use_negative:
- Data lack regularly injected pooled QC samples measured during one or more sequences
  — the control-chart logic depends on QCpool run-order series.
- Raw instrument files that have not been integrated and exported via Sciex MultiQuant
  to txt format — QComics expects the exported descriptor table, not vendor binaries.
- User intends to assess batch drift or missing-value handling separately — the article
  treats those as distinct sequential QC steps rather than parts of this charting
  step.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: Shiny app that renders the QCpool run-sequence control charts and applies
    Westgard rules to flag out-of-control samples
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment; the app is launched via QComics::run_app() from RStudio
    or a shiny server
- name: ggplot2
  role: Graphics backend used to draw the descriptor control charts with SD-based
    control limits
- name: Sciex MultiQuant
  role: Upstream software that integrates the targeted assay and exports the QCpool
    descriptor table as txt for QComics input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:66581f4306cce9d1f774425d177197cf439b3ae71a431be70635e072fe617283
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - Sciex MultiQuant
  - dplyr
  - ggplot2
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Run-sequence quality charting

## Summary

Plot each chemical descriptor of the pooled QC (QCpool) sample across injection order with control limits at ±1SD, ±2SD and ±3SD and apply Westgard rules to flag 'out-of-control' observations, as implemented in the QComics Shiny app (R/ggplot2).

## When to use

Use when targeted metabolomics or lipidomics sequences contain regularly injected QCpool samples exported from Sciex MultiQuant (> v3.0.3) as txt files, and you need to detect drifts, contaminants, or out-of-control measurements before further QC steps.

## When NOT to use

- Data lack regularly injected pooled QC samples measured during one or more sequences — the control-chart logic depends on QCpool run-order series.
- Raw instrument files that have not been integrated and exported via Sciex MultiQuant to txt format — QComics expects the exported descriptor table, not vendor binaries.
- User intends to assess batch drift or missing-value handling separately — the article treats those as distinct sequential QC steps rather than parts of this charting step.

## Inputs

- QCpool chemical descriptor table (data_df) exported from Sciex MultiQuant (> v3.0.3) as txt
- Injection sequence/run order for QCpool samples
- Descriptor-wise mean and SD computed across QCpool injections

## Outputs

- Per-descriptor run-sequence control chart figures with ±1SD, ±2SD, ±3SD control limits
- Westgard violation table (injection–descriptor pairs plus triggering rule)
- Formatted QC table of data_df by sequence (make_nice_table output)

## How to apply

Load the QCpool chemical-descriptor series (data_df) together with its injection sequence, compute the mean and SD of every descriptor across QCpool injections, and for each descriptor draw a run-sequence control chart (make_qc_graph(data_df, yaxis, sequence)) with horizontal limits at ±1SD, ±2SD and ±3SD around the QCpool mean. Evaluate the four Westgard criteria on each descriptor series: a single point beyond ±3SD; two consecutive points beyond ±2SD on the same side of the mean; four consecutive points beyond ±1SD on the same side; and ten consecutive points on the same side of the mean. Record every injection-descriptor pair that violates any rule and note which rule triggered; when criteria are not met, neighbouring study samples are scrutinised for possible discard and reanalysis. Render one control-chart figure per descriptor and export the flagged violations as a tabulated table (make_nice_table(data_df, sequence)). QComics was developed to be implementable without advanced statistical or programming skills, and its quality criteria were adapted from bioanalytical method validation guidelines and existing metabolomics QC literature.

## Related tools

- **QComics** (Shiny app that renders the QCpool run-sequence control charts and applies Westgard rules to flag out-of-control samples) — https://github.com/ricoderks/QComics
- **R** (Runtime environment; the app is launched via QComics::run_app() from RStudio or a shiny server)
- **ggplot2** (Graphics backend used to draw the descriptor control charts with SD-based control limits)
- **Sciex MultiQuant** (Upstream software that integrates the targeted assay and exports the QCpool descriptor table as txt for QComics input)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The R script runs without error and produces one figure file per descriptor.
- The exported violation table lists injection–descriptor pairs with the triggering Westgard rule for each.
- Violation flags agree with an expert-reviewed reference set of out-of-control observations.
- Control limits on the plots visually match the QCpool mean ± 1SD, ±2SD and ±3SD lines.
- QComics is installed from GitHub (remotes::install_github("ricoderks/QComics")) and the Shiny app launches via QComics::run_app().

## Limitations

- QComics is marked as experimental (lifecycle badge) — the charting logic may change between versions.
- The article does not report quantitative benchmark or validation metrics for this specific step; reliability is asserted via prior applications in metabolomics and metallomics studies.
- Criteria require QCpool injections to be evenly distributed across the sequence; uneven spacing or too few QCpool points can distort consecutive-point Westgard checks.

## Evidence

- [results] control limits and Westgard rules on QCpool series: [evidence span withheld — anchor status: not_found]
- [abstract] QComics protocol step: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations"
- [readme] installation command: "remotes::install_github("ricoderks/QComics")"
