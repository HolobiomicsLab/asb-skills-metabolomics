---
name: parametric-t-test-anova-analysis
description: Use when you have a preprocessed, normalized feature matrix with group
  labels (two groups or more) and need per-feature classical significance testing
  with multiple-testing correction across the many tests.
when_to_use_negative:
- When group labels are absent and the goal is unsupervised EDA (PCA, clustering)
  rather than per-feature significance testing
- When the feature matrix has not yet gone through missing-value imputation and normalization
  — applying t-test/ANOVA directly to raw features would violate the sequential POMAShiny
  workflow order (upload → preprocessing → EDA → statistical analysis)
- When a large proportion of features fail normality/homoscedasticity checks and the
  non-parametric Mann-Whitney U/Kruskal-Wallis alternatives are the correct panel
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: Web-based panel that exposes the univariate t-test/ANOVA analyses on uploaded
    feature/metadata CSVs
  repo: github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the univariate t-test/ANOVA functions and
    FDR correction used outside or inside the web app
  repo: github.com/nutrimetabolomics/POMA
- name: R
  role: Language in which POMA/POMAShiny statistical tests are implemented and from
    which the results table is compiled
- name: tidyverse
  role: Used to assemble the per-feature statistic/p-value/adjusted-p-value results
    table after the tests run
- name: MSnbase
  role: Back-end R package supporting the POMAShiny data structures into which the
    feature matrix and metadata are loaded
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:9dffd3e182b6af3ee3c662b01e006d7218a61e11f009671bd101cdef7b2ef135
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# parametric-t-test-anova-analysis

## Summary

Run POMAShiny/POMA univariate parametric testing — t-test for two-group contrasts and ANOVA for multi-group designs — on metabolomics/proteomics feature tables (e.g. ST000284 ST000336), collecting per-feature statistics, raw and FDR-adjusted p-values into a single results table.

## When to use

Use when you have a preprocessed, normalized feature matrix with group labels (two groups or more) and need per-feature classical significance testing with multiple-testing correction across the many tests.

## When NOT to use

- When group labels are absent and the goal is unsupervised EDA (PCA, clustering) rather than per-feature significance testing
- When the feature matrix has not yet gone through missing-value imputation and normalization — applying t-test/ANOVA directly to raw features would violate the sequential POMAShiny workflow order (upload → preprocessing → EDA → statistical analysis)
- When a large proportion of features fail normality/homoscedasticity checks and the non-parametric Mann-Whitney U/Kruskal-Wallis alternatives are the correct panel

## Inputs

- Preprocessed feature matrix (CSV features file) from POMAShiny data upload/preprocessing panels
- Target (metadata) file assigning samples to two or more study groups
- POMA SummarizedExperiment object built from the above two files

## Outputs

- Per-feature results table with test statistic, raw p-value and FDR-adjusted p-value
- Visual/interactive display of significant features from the POMAShiny statistical analysis panel

## How to apply

Load the preprocessed feature matrix and its target (metadata) file as a POMA SummarizedExperiment object in R. For a two-group contrast (e.g. ST000336 design) use the t-test; for a three-group or larger design (e.g. ST000284) use ANOVA as offered by POMAShiny's statistical analysis panel. POMA's univariate functions compute these per-feature tests and, because a large number of tests is performed, produce FDR-adjusted p-values alongside raw ones. Compile test statistics, raw p-values and adjusted p-values for every feature into one results table with tidyverse, then validate that each row has non-empty statistic, p-value and adjusted-p columns and that the group structure was respected. If a parametric assumption check fails, fall back to the Mann-Whitney U / Kruskal-Wallis alternatives in the same panel rather than forcing the t-test/ANOVA.

## Related tools

- **POMAShiny** (Web-based panel that exposes the univariate t-test/ANOVA analyses on uploaded feature/metadata CSVs) — github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the univariate t-test/ANOVA functions and FDR correction used outside or inside the web app) — github.com/nutrimetabolomics/POMA
- **R** (Language in which POMA/POMAShiny statistical tests are implemented and from which the results table is compiled)
- **tidyverse** (Used to assemble the per-feature statistic/p-value/adjusted-p-value results table after the tests run)
- **MSnbase** (Back-end R package supporting the POMAShiny data structures into which the feature matrix and metadata are loaded)

## Examples

```
shiny::runApp(appDir = "app")  # then, in the POMAShiny statistical analysis panel, select T-test/ANOVA on the preprocessed ST000284 feature matrix
```

## Evaluation signals

- The results table contains one row per feature with non-empty statistic, raw p-value and FDR-adjusted p-value columns
- The FDR-adjusted p-values are present for every tested feature, confirming POMA applied the multiple-testing correction it is designed to apply
- The group contrast matches the metadata: t-test used only where the target file has two groups, ANOVA where three or more (e.g. ST000284 three-group design)
- Every tested feature was drawn from the matrix that already passed the missing-value filter (features with >20% missing values per group removed by default) and normalization steps, not the raw upload
- Re-running the analysis outside the web interface via the POMA package reproduces the same statistics and adjusted p-values (the POMA/POMAShiny split is designed for reproducibility)

## Limitations

- Multiple-testing correction is applied across a large number of per-feature tests, so adjusted p-values can be conservative for weak signals
- Parametric tests assume adequate distributional behaviour of each feature; if assumptions are violated, the Mann-Whitney U / Kruskal-Wallis alternatives in the same panel should be used instead
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions of bs4Dash due to substantial breaking changes in the API

## Evidence

- [results] POMAShiny's univariate analysis (T-test/ANOVA and Mann-Whitney U/Kruskal-Wallis) computes FDR-adjusted p-values, because a large number of tests is performed in these analyses.: [evidence span withheld — anchor status: not_found]
- [readme] POMAShiny workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels.: "POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels."
- [results] users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default): "users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [intro] including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
