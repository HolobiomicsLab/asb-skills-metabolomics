---
name: metabolomics-qcpool-series-monitoring
description: Use when a targeted metabolomics or lipidomics study has pooled QC samples
  (QCpool) measured at regular intervals across one or more sequences, and you need
  to detect signal drift, background/carryover effects or out-of-control injections
  before downstream missing-value handling and outlier removal.
when_to_use_negative:
- The dataset lacks pooled QC samples injected at regular intervals — QComics' QCpool-based
  monitoring cannot be performed and preanalytical quality markers should be used
  instead.
- The analysis goal is handling missing values versus truly absent data, removing
  outlying samples, or monitoring preanalytical quality markers — these are later
  steps of the QComics workflow, not the run-sequence control-chart step.
- Data are not from MS-based metabolomics/lipidomics injections exported via Sciex
  MultiQuant and you cannot reformat them as the QCpool descriptor txt table expected
  by the R app.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: R Shiny app that implements the QCpool run-sequence control charts, Westgard
    rule evaluation and violation reporting for this skill
  repo: https://github.com/ricoderks/QComics
- name: R
  role: runtime in which the QComics Shiny app is launched (QComics::run_app())
- name: RStudio
  role: environment for running the app locally via QComics::run_app()
- name: ggplot2
  role: used by make_qc_graph(data_df, yaxis, sequence) to draw the ±1/2/3SD run-sequence
    control charts
- name: Microsoft Excel
  role: alternative software for performing this QC step without advanced statistical
    skills
- name: MetaboAnalyst
  role: alternative webtool for performing this QC step without advanced statistical
    skills
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:04a6b50eeb6e14f8e7b08e3849390a848dbaf4555fe994968c1621123fa273d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - MetaboAnalyst
  - Microsoft Excel
  - QComics
  - R
  - RStudio
  - dplyr
  - ggplot2
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# metabolomics-qcpool-series-monitoring

## Summary

Monitors pooled QC (QCpool) metabolomics series by plotting each chemical descriptor over run order with Shewhart control limits at ±1SD, ±2SD and ±3SD, then applying Westgard decision rules to flag out-of-control injections and decide whether a sequence is acceptable. This is the drift/contamination detection stage of the QComics protocol (Anal. Chem. 2024).

## When to use

Use when a targeted metabolomics or lipidomics study has pooled QC samples (QCpool) measured at regular intervals across one or more sequences, and you need to detect signal drift, background/carryover effects or out-of-control injections before downstream missing-value handling and outlier removal.

## When NOT to use

- The dataset lacks pooled QC samples injected at regular intervals — QComics' QCpool-based monitoring cannot be performed and preanalytical quality markers should be used instead.
- The analysis goal is handling missing values versus truly absent data, removing outlying samples, or monitoring preanalytical quality markers — these are later steps of the QComics workflow, not the run-sequence control-chart step.
- Data are not from MS-based metabolomics/lipidomics injections exported via Sciex MultiQuant and you cannot reformat them as the QCpool descriptor txt table expected by the R app.

## Inputs

- QCpool chemical descriptor table (data_df) exported from Sciex MultiQuant (> v3.0.3) as txt
- injection sequence/run order for each QCpool sample

## Outputs

- one Shewhart run-sequence control-chart figure per chemical descriptor
- westgard_violations.csv listing each injection–descriptor pair that violates a Westgard rule, with the triggering rule
- accept/reject decision per batch, plus identification of neighboring study samples for possible discard and reanalysis

## How to apply

Import the QCpool chemical-descriptor table (data_df) exported from Sciex MultiQuant (> v3.0.3) as txt, with each descriptor ordered by injection sequence. For every descriptor compute the QCpool mean and SD, then build a run-sequence Shewhart chart (make_qc_graph(data_df, yaxis, sequence) in R/ggplot2) with upper and lower limits at ±1SD, ±2SD and ±3SD around the mean. Evaluate each descriptor series against the four Westgard rules — a single point beyond ±3SD; two consecutive points beyond ±2SD on the same side of the mean; four consecutive points beyond ±1SD on the same side; ten consecutive points on the same side of the mean — recording each injection–descriptor violation and the triggering rule. When Westgard criteria are not met, scrutinize neighboring study samples for possible discard and reanalysis. Render one control-chart figure per descriptor and export the flagged pairs as westgard_violations.csv via make_nice_table(data_df, sequence).

## Related tools

- **QComics** (R Shiny app that implements the QCpool run-sequence control charts, Westgard rule evaluation and violation reporting for this skill) — https://github.com/ricoderks/QComics
- **R** (runtime in which the QComics Shiny app is launched (QComics::run_app()))
- **RStudio** (environment for running the app locally via QComics::run_app())
- **ggplot2** (used by make_qc_graph(data_df, yaxis, sequence) to draw the ±1/2/3SD run-sequence control charts)
- **Microsoft Excel** (alternative software for performing this QC step without advanced statistical skills)
- **MetaboAnalyst** (alternative webtool for performing this QC step without advanced statistical skills)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_in_browser()
```

## Evaluation signals

- Every chemical descriptor yields a run-sequence control chart with control limits drawn at exactly ±1SD, ±2SD and ±3SD of the QCpool mean.
- westgard_violations.csv contains one row per injection–descriptor pair flagged by any of the four Westgard rules, and the triggering rule is recorded for each violation.
- The R script executes without error: QComics::run_app() (or run_in_browser()) completes, per-descriptor figure files are produced, and westgard_violations.csv exists.
- Violation flags agree with an expert-reviewed reference set of out-of-control injections.
- When Westgard criteria are not met, neighboring study samples have been explicitly scrutinized and a discard/reanalysis decision recorded.

## Limitations

- The QComics package is marked experimental and provides no changelog; version number, license, installation instructions and dependency requirements are not stated in the article text.
- Quality criteria are adapted from bioanalytical method validation guidelines rather than universally standardized, so pre-existing subjectivity and poor inter-laboratory harmonization may persist.
- Figures 2–4 and citations 43–50 supporting prior applications are not present in the provided text, so visual/manual validation of control charts is not fully reproducible from this excerpt alone.

## Evidence

- [abstract] plot each chemical descriptor over run order with upper and lower limits at ±SD, ±2SD, and ±3SD: [evidence span withheld — anchor status: not_found]
- [methods] applying Westgard control rules: "From Rstudio run the app as"
- [intro] QComics flags 'out-of-control' QC samples by plotting each chemical descriptor over run order: "this QC protocol can easily be performed in software that is available to most researchers (e.g., Microsoft Excel and MetaboAnalyst webtool)"
- [readme] Sciex Multiquant (> v3.0.3) software and exported to `txt` format: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
- [readme] make_qc_graph(data_df, yaxis, sequence) in QComics/ggplot2: "remotes::install_github("ricoderks/QComics")"
