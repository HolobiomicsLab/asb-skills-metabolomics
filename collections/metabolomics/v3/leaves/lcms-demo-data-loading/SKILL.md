---
name: lcms-demo-data-loading
description: Use when validating or demonstrating MetaboReport end-to-end without
  user-supplied data — e.g. reproducing the File S5 example HTML report or testing
  that preprocessing, statistics, visualisation and reporting modules communicate
  via new_app_state().
when_to_use_negative:
- You have your own LC-MS peak/feature table to analyse — load it directly instead
  of the demo dataset
- You only need the report-generation functions without running the app workflow
- The goal is non-LC-MS metabolomics data types not covered by the bundled cancerCell
  demo
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboReport
  role: R Shiny app whose run_app() entry point hosts the modules and whose package
    bundles the cancerCell demo LC-MS dataset
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment; package installed via remotes::install_github() and started
    with run_app()
- name: Shiny
  role: Framework for the modular app whose modules share the loaded dataset through
    the new_app_state() object
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:a572cad97cc60de40836a48cca4ccc03efb9ba0c13e7d8fa5c4fdd346d83ab8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - Shiny
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lcms-demo-data-loading

## Summary

Loads the bundled cancerCell demo LC-MS dataset into the MetaboReport R Shiny app so that the preprocessing, statistical analysis, visualisation and reporting modules have data to operate on. It is the standard end-to-end smoke test of the app's shared-state workflow.

## When to use

Use when validating or demonstrating MetaboReport end-to-end without user-supplied data — e.g. reproducing the File S5 example HTML report or testing that preprocessing, statistics, visualisation and reporting modules communicate via new_app_state().

## When NOT to use

- You have your own LC-MS peak/feature table to analyse — load it directly instead of the demo dataset
- You only need the report-generation functions without running the app workflow
- The goal is non-LC-MS metabolomics data types not covered by the bundled cancerCell demo

## Inputs

- Bundled cancerCell (MetaboReport) demo LC-MS dataset shipped inside the R package
- MetaboReport Shiny app instance started via run_app(), with shared state from new_app_state()

## Outputs

- Demo LC-MS dataset resident in the app's shared state object, ready for the preprocessing and statistical analysis modules
- Downstream interactive plots/tables and a self-contained HTML report generated from the demo data

## How to apply

Install MetaboReport as an R package with remotes::install_github("YonghuiDong/MetaboReport") (or download the standalone app from SourceForge, which the authors recommend as the easiest option), then start the Shiny app with library(MetaboReport); run_app() or MetaboReport::run_app(). Inside the app, load the bundled cancerCell (MetaboReport) demo LC-MS dataset rather than uploading external files. The dataset flows into the shared state object created by new_app_state(), through which the R/mod_*.R Shiny modules (preprocessing, statistical analysis, visualisation) exchange results. Once loaded, run the preprocessing, statistical analysis and visualisation modules in order; their outputs accumulate in the state and feed the final self-contained HTML report generation.

## Related tools

- **MetaboReport** (R Shiny app whose run_app() entry point hosts the modules and whose package bundles the cancerCell demo LC-MS dataset) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment; package installed via remotes::install_github() and started with run_app())
- **Shiny** (Framework for the modular app whose modules share the loaded dataset through the new_app_state() object)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- The Shiny app launches without error after library(MetaboReport); run_app() and exposes the demo dataset loader
- The cancerCell demo LC-MS dataset appears in the app and the preprocessing, statistical analysis and visualisation modules all execute on it without errors
- The exported report.html is a non-empty, self-contained HTML document containing project details, interactive plots/tables, summary statistics and explanations of results
- The generated report mirrors the example reports provided in the paper's File S5
- testthat::test_local() regression tests in tests/testthat/ pass on the installed package

## Limitations

- The public package API is intentionally small (run_app()); analysis helpers are internal, so dataset handling cannot be scripted outside the app without relying on undocumented internals
- The cancerCell dataset is a demo LC-MS dataset — results are illustrative, not a substitute for real project data
- The README notes a fixed report-generation bug and only recently added Chinese report generation in v0.3.6, so older versions may produce incomplete reports

## Evidence

- [supplementary] Load the bundled cancerCell demo LC-MS dataset into the app.: [evidence span withheld — anchor status: not_found]
- [readme] Modules communicate through the state object created by new_app_state().: "Modules communicate through the state object created by `new_app_state()`."
- [readme] After installation, you can use library(MetaboReport); run_app() to start the app.: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [abstract] simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting: "simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [methods] This is the easiest way to install and use MetaboReport (recommended): "This is the easiest way to install and use MetaboReport (recommended)"
