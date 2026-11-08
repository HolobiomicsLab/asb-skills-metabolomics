---
name: metabolomics-summary-statistics-visualization
description: Use when a preprocessed metabolomics feature/sample table needs exploratory
  visualisation and summary statistics, and the results must be packaged into a self-contained
  HTML report for a lab or core facility project.
when_to_use_negative:
- The goal is raw LC-MS file conversion or peak detection — MetaboReport starts from
  data already loaded for preprocessing, not vendor raw acquisition files
- 'You need a scriptable, non-interactive pipeline: the public API is intentionally
  limited to run_app(), so analysis helpers are internal and not meant for programmatic
  reuse'
- The deliverable is only a static figure outside the report context — the skill's
  value is the integrated HTML report, not standalone plotting
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3656
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: Shiny web app whose visualisation and reporting modules produce interactive
    plots, tables, summary statistics, and the comprehensive HTML report
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R Shiny
  role: framework in which MetaboReport's modular visualisation/reporting modules
    are written (golem schema)
- name: ggplot2
  role: imported plotting dependency underpinning the plotting/report functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:5fc6568816fa144c40031c97091f5c84c89f1f378b13508ed976ad5232527eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - R Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3656
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-biomarker-visualization
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolomics-summary-statistics-visualization

## Summary

Generates interactive plots, tables, and summary statistics from preprocessed metabolomics peak matrices inside MetaboReport, a modular R Shiny app, and folds them into a comprehensive self-contained HTML report.

## When to use

Use when a preprocessed metabolomics feature/sample table needs exploratory visualisation and summary statistics, and the results must be packaged into a self-contained HTML report for a lab or core facility project.

## When NOT to use

- The goal is raw LC-MS file conversion or peak detection — MetaboReport starts from data already loaded for preprocessing, not vendor raw acquisition files
- You need a scriptable, non-interactive pipeline: the public API is intentionally limited to run_app(), so analysis helpers are internal and not meant for programmatic reuse
- The deliverable is only a static figure outside the report context — the skill's value is the integrated HTML report, not standalone plotting

## Inputs

- preprocessed metabolomics peak/feature matrix (e.g. the bundled cancerCell LC-MS dataset)
- sample metadata and project details entered through the Shiny app
- outputs of upstream preprocessing and statistical analysis modules via the shared new_app_state() state object

## Outputs

- interactive plots of feature/sample summaries
- interactive tables and summary statistics of statistical results
- self-contained HTML report including project details, an introduction, plots/tables, statistical results, and in-depth explanations/interpretation

## How to apply

Install MetaboReport from GitHub with remotes::install_github("YonghuiDong/MetaboReport") and launch it via library(MetaboReport); run_app(). Load a metabolomics dataset (e.g. the bundled cancerCell demo LC-MS dataset) into the app, then run the preprocessing and statistical analysis modules first, since Shiny modules share results through the state object created by new_app_state(). Next, execute the visualisation module to produce interactive plots and tables together with summary statistics over the analysed features and samples. All analysis, plotting, and report functions live in R/fct_*.R, with plotting built on ggplot2 imports, while the report module assembles project details, an introduction, the interactive plots/tables, statistical results, and in-depth explanations into one HTML document. A Chinese-language report can also be generated per the v0.3.6 release notes.

## Related tools

- **MetaboReport** (Shiny web app whose visualisation and reporting modules produce interactive plots, tables, summary statistics, and the comprehensive HTML report) — https://github.com/YonghuiDong/MetaboReport
- **R Shiny** (framework in which MetaboReport's modular visualisation/reporting modules are written (golem schema))
- **ggplot2** (imported plotting dependency underpinning the plotting/report functions)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- The generated report.html is a non-empty, self-contained HTML document (mirrors the File S5 example reports)
- The report contains all expected sections: project details, an introduction, interactive plots and tables, statistical results, and in-depth explanations/interpretation
- The plots and tables shown match the statistical analysis results held in the shared state object, confirming module hand-off via new_app_state()
- testthat::test_local() regression tests in tests/testthat/ pass after any customisation of plotting/report functions
- Where applicable, the alternative Chinese-language report is generated with the same content structure

## Limitations

- The public package API is intentionally small (only run_app()); analysis and report helpers are internal and may change without a compatibility contract
- Report generation had bug fixes as recent as v0.3.6, so report content/output should be validated per release
- Modules depend on the shared state object created by new_app_state(); running the visualisation module without upstream preprocessing/statistical modules leaves it without inputs

## Evidence

- [abstract] Comprehensive HTML report content: "MetaboReport produces a comprehensive HTML report, including project details, an introduction, interactive plots and tables, statistical results and an in-depth explanations and interpretation of the"
- [abstract] Modular Shiny architecture: "MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development"
- [readme] Plotting and report functions location: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report functions."
- [readme] Module communication via state object: "`R/mod_*.R` contains Shiny modules. Modules communicate through the state
  object created by `new_app_state()`."
- [readme] Report generation updates in v0.3.6: "- Bug fixed in report generation
- Report content generated
- Allow to generate Chinese report"
