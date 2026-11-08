---
name: shiny-app-workflow-execution
description: Use when an agent must execute an interactive metabolomics analysis workflow
  inside a Shiny app (MetaboReport), passing an LC-MS dataset through preprocessing,
  statistical analysis and visualisation modules before report generation.
when_to_use_negative:
- The task is scripted/command-line metabolomics processing — MetaboReport exposes
  only run_app() as its public API, so automated pipelines cannot call its internal
  analysis helpers directly.
- The dataset is already preprocessed and analysed and only the HTML report needs
  to be regenerated from prior results.
- A non-interactive batch tool is required and GUI interaction is impossible in the
  execution environment.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: The Shiny app whose modules (preprocessing, exploration, statistical analysis,
    visualisation) are executed in sequence over the shared new_app_state() object
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R Shiny
  role: Framework providing the modular interactive app structure (golem schema) in
    which the workflow modules run
  repo: https://github.com/YonghuiDong/MetReport
- name: ggplot2
  role: Imported plotting library used by the app's visualisation module during workflow
    execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:18017ddac63bf3ff6283c66ca450b030e5ba49c96c92820700688aeddea7ce27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - R Shiny
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# shiny-app-workflow-execution

## Summary

Run the MetaboReport R Shiny app end-to-end — loading an LC-MS metabolomics dataset, executing its preprocessing, statistical analysis and visualisation modules, and letting the modules share results through the state object created by new_app_state() to enable downstream reporting.

## When to use

Use when an agent must execute an interactive metabolomics analysis workflow inside a Shiny app (MetaboReport), passing an LC-MS dataset through preprocessing, statistical analysis and visualisation modules before report generation.

## When NOT to use

- The task is scripted/command-line metabolomics processing — MetaboReport exposes only run_app() as its public API, so automated pipelines cannot call its internal analysis helpers directly.
- The dataset is already preprocessed and analysed and only the HTML report needs to be regenerated from prior results.
- A non-interactive batch tool is required and GUI interaction is impossible in the execution environment.

## Inputs

- Metabolomics (LC-MS) dataset, e.g. the bundled cancerCell demo dataset
- MetaboReport package installed from GitHub or the standalone SourceForge app
- R environment with Shiny, golem, and ggplot2 (package Imports)

## Outputs

- Completed in-app workflow state (preprocessing, statistical analysis, and visualisation results) available for HTML report generation
- Interactive plots and tables produced by the visualisation module

## How to apply

Install MetaboReport from GitHub with remotes::install_github("YonghuiDong/MetaboReport") (or download the standalone app from SourceForge), then start it with library(MetaboReport); run_app(). Load the metabolomics dataset — e.g. the bundled cancerCell demo LC-MS dataset — into the app UI. Execute the modules in order: preprocessing, exploration, statistical analysis, and visualisation; the app is written modularly following the golem schema, and R/mod_*.R modules communicate through the shared state object created by new_app_state(), so results from earlier modules feed later ones automatically. Because the public API is intentionally just run_app(), analysis helpers (R/fct_*.R) are internal and should be invoked via the GUI rather than called directly. Once all modules have run, the shared state is ready for comprehensive HTML report generation.

## Related tools

- **MetaboReport** (The Shiny app whose modules (preprocessing, exploration, statistical analysis, visualisation) are executed in sequence over the shared new_app_state() object) — https://github.com/YonghuiDong/MetaboReport
- **R Shiny** (Framework providing the modular interactive app structure (golem schema) in which the workflow modules run) — https://github.com/YonghuiDong/MetReport
- **ggplot2** (Imported plotting library used by the app's visualisation module during workflow execution)

## Examples

```
Rscript -e 'remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()'
```

## Evaluation signals

- run_app() launches the app without errors after library(MetaboReport) (or MetaboReport::run_app())
- The bundled cancerCell demo LC-MS dataset loads successfully into the app UI
- All modules (preprocessing, statistical analysis, visualisation) execute and pass results through the new_app_state() shared state without errors
- package checks pass: testthat::test_local() and rcmdcheck::rcmdcheck(args = c("--no-manual", "--no-build-vignettes"))
- Downstream HTML report generation succeeds, indicating the workflow state was fully populated

## Limitations

- The public package API is intentionally small (only run_app()); analysis helpers are internal, so their implementation can evolve without creating an accidental compatibility contract — scripts depending on internal functions may break.
- Release notes note past bugs in report generation and a bug fix for report generation (v0.3.6), so report output should be verified after workflow execution.
- The R-package install path requires remotes and GitHub access; a standalone Windows download from SourceForge is offered as the easiest alternative.

## Evidence

- [abstract] MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development: "MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development"
- [methods] run_app() to start the app: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app"
- [readme] modules communicate through the state object: "`R/mod_*.R` contains Shiny modules. Modules communicate through the state object created by `new_app_state()`."
- [readme] public API is intentionally small: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve"
- [abstract] simplifies metabolomics experiment design through reporting: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
