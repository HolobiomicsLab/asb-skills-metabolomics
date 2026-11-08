---
name: metabolomics-html-report-generation
description: Use when a metabolomics LC-MS dataset has been preprocessed, statistically
  analysed and visualised (e.g. the bundled cancerCell demo in MetaboReport) and the
  results must be delivered as one self-contained HTML document to lab members or
  service clients.
when_to_use_negative:
- No analysis session has been run yet — the report is generated from the shared state
  populated by the preprocessing/statistics/visualisation modules, so invoking it
  on an empty state yields no meaningful content
- The deliverable must be a raw machine-readable results table rather than a human-facing
  narrative HTML report
- The target environment cannot run an R/Shiny app (MetaboReport is distributed as
  an R package or standalone desktop download)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaboReport
  role: Shiny app whose report module composes the self-contained HTML report from
    the shared analysis state
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R Shiny
  role: Web framework in which the modular preprocessing/analysis/reporting app is
    written (golem schema)
- name: golem
  role: Production-grade Shiny app framework governing the app_*/mod_* structure used
    to assemble the reporting pipeline
- name: ggplot2
  role: Rendering engine for the plots embedded in the HTML report
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:aa15c2aac570c9682c40893129f6ef38346a1c458b705741ce0dd7e8b0ea5b4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - R Shiny
  - ggplot2
  - golem
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-html-report-generation

## Summary

Generate a single self-contained HTML report from a completed metabolomics analysis session using MetaboReport, bundling project details, an introduction, interactive plots and tables, summary statistics, and in-depth explanations of the statistical results.

## When to use

Use when a metabolomics LC-MS dataset has been preprocessed, statistically analysed and visualised (e.g. the bundled cancerCell demo in MetaboReport) and the results must be delivered as one self-contained HTML document to lab members or service clients.

## When NOT to use

- No analysis session has been run yet — the report is generated from the shared state populated by the preprocessing/statistics/visualisation modules, so invoking it on an empty state yields no meaningful content
- The deliverable must be a raw machine-readable results table rather than a human-facing narrative HTML report
- The target environment cannot run an R/Shiny app (MetaboReport is distributed as an R package or standalone desktop download)

## Inputs

- Processed LC-MS metabolomics feature table loaded into the MetaboReport app (e.g. the bundled cancerCell demo dataset)
- Completed preprocessing, statistical analysis and visualisation state held in the new_app_state() shared object
- Project metadata entered in the app (project details section of the report)

## Outputs

- A single self-contained report.html file with project details, introduction, interactive plots and tables, summary statistics and in-depth explanations of the results
- Optionally a Chinese-language variant of the same report (v0.3.6+)

## How to apply

Install MetaboReport from GitHub with remotes::install_github("YonghuiDong/MetaboReport") and launch the Shiny app via library(MetaboReport); run_app(). Load the LC-MS dataset (e.g. the bundled cancerCell demo) and execute the preprocessing, statistical analysis and visualisation modules in sequence; the modules communicate through the shared state object created by new_app_state(), so each module's outputs feed downstream report content. Trigger report generation after the analysis modules complete, and optionally select the Chinese-language report (supported since v0.3.6). The output report.html should be a non-empty self-contained HTML document containing project details, an introduction, the interactive plots and tables generated during the session, summary statistics, and in-depth explanations and interpretation of the results.

## Related tools

- **MetaboReport** (Shiny app whose report module composes the self-contained HTML report from the shared analysis state) — https://github.com/YonghuiDong/MetaboReport
- **R Shiny** (Web framework in which the modular preprocessing/analysis/reporting app is written (golem schema))
- **golem** (Production-grade Shiny app framework governing the app_*/mod_* structure used to assemble the reporting pipeline)
- **ggplot2** (Rendering engine for the plots embedded in the HTML report)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- report.html exists, is non-empty, and is a self-contained HTML document (no external dependencies required to view it)
- Report contains all required sections: project details, an introduction, interactive plots and tables, summary statistics, and in-depth explanations/interpretation of results
- Report content matches the example reports provided as File S5 (the demo cancerCell LC-MS case)
- Plots/tables in the report reflect the actual state produced by the preprocessing, statistical analysis and visualisation modules in the same session
- Package-level automated regression tests in tests/testthat/ pass (testthat::test_local())

## Limitations

- The public package API is intentionally limited to run_app(); report and analysis helpers are internal, so report behaviour can change between versions without notice
- Report generation had bugs fixed in v0.3.6, implying earlier versions may produce incomplete reports
- Report content is tied to what was computed in the app session — sections omitted from the analysis modules (e.g. skipped statistical analysis) will be missing from the report

## Evidence

- [abstract] MetaboReport produces a comprehensive HTML report, including project details, an introduction, interactive plots and tables, statistical results and an in-depth explanations: "MetaboReport produces a comprehensive HTML report, including project details, an introduction, interactive plots and tables, statistical results and an in-depth explanations"
- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [methods] you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app"
- [readme] R/mod_*.R contains Shiny modules. Modules communicate through the state object created by new_app_state().: "- `R/mod_*.R` contains Shiny modules. Modules communicate through the state
  object created by `new_app_state()`."
