---
name: metabolite-feature-table-statistical-testing
description: Use when you have a preprocessed metabolomics feature table (e.g. ST000284
  or ST000336 from Metabolomics Workbench) with a two- or three-group target/metadata
  design and need per-feature test statistics, raw p-values, and FDR-adjusted p-values.
when_to_use_negative:
- Input is still raw or unnormalized feature data — POMAShiny's workflow puts imputation,
  normalization, and outlier detection (e.g. 20% missing-value removal threshold)
  in the pre-processing panel before any statistical testing, so this skill should
  not be applied before those steps.
- The design is not a simple two- or three-group contrast — POMAShiny's univariate
  panel is built around two-group (t-test, Mann-Whitney) and three-group (ANOVA, Kruskal-Wallis)
  comparisons, so other designs (e.g. paired, repeated-measures, or regression-style
  models) fall outside its scope.
- A single overall omnibus hypothesis is needed rather than a battery of per-feature
  tests — the FDR adjustment assumes many independent per-feature tests, so this skill
  is inappropriate when only a few pre-specified features are of interest.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: POMAShiny
  role: Web-based front-end that hosts the univariate statistical analysis panel (t-test/ANOVA
    and Mann-Whitney/Kruskal-Wallis) with FDR correction
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the same univariate tests outside the web
    environment, on SummarizedExperiment objects
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: R packages used to compile test statistics, raw p-values, and adjusted p-values
    into one results table
- name: R
  role: Execution environment for the POMA/POMAShiny univariate testing workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:d0666d31845e9fb6bd9c17c33ea4468952614d1dfe42049fa3211e20cb9daf13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolite-feature-table-statistical-testing

## Summary

Run univariate statistical tests (parametric and non-parametric) across metabolite features in a preprocessed feature table and correct the resulting p-values for multiple testing. Grounded in POMAShiny/POMA's statistical analysis panel, which computes FDR-adjusted p-values because a large number of tests is performed.

## When to use

Use when you have a preprocessed metabolomics feature table (e.g. ST000284 or ST000336 from Metabolomics Workbench) with a two- or three-group target/metadata design and need per-feature test statistics, raw p-values, and FDR-adjusted p-values.

## When NOT to use

- Input is still raw or unnormalized feature data — POMAShiny's workflow puts imputation, normalization, and outlier detection (e.g. 20% missing-value removal threshold) in the pre-processing panel before any statistical testing, so this skill should not be applied before those steps.
- The design is not a simple two- or three-group contrast — POMAShiny's univariate panel is built around two-group (t-test, Mann-Whitney) and three-group (ANOVA, Kruskal-Wallis) comparisons, so other designs (e.g. paired, repeated-measures, or regression-style models) fall outside its scope.
- A single overall omnibus hypothesis is needed rather than a battery of per-feature tests — the FDR adjustment assumes many independent per-feature tests, so this skill is inappropriate when only a few pre-specified features are of interest.

## Inputs

- Preprocessed metabolomics feature matrix (CSV, one column per metabolite feature, one row per sample) from e.g. ST000284 or ST000336
- Target/metadata CSV file mapping samples to two- or three-group study conditions
- POMA SummarizedExperiment (or POMAShiny-compatible) R object holding features and target together

## Outputs

- Per-feature results table (tidyverse data frame) with columns: feature identifier, test statistic, raw p-value, FDR-adjusted p-value
- Separate result sets for the parametric (t-test/ANOVA) and non-parametric (Mann-Whitney/Kruskal-Wallis) alternatives

## How to apply

Load the preprocessed feature matrix and its target (metadata) CSV files into R as a POMA SummarizedExperiment object (POMA's home format, also compatible with MSnbase). For each metabolite feature, run the tests offered by POMAShiny's statistical analysis panel: t-test for two-group contrasts and ANOVA for the three-group design as the parametric options, Mann-Whitney U (two groups) and Kruskal-Wallis (three groups) as the non-parametric alternatives. Because POMAShiny computes FDR-adjusted p-values whenever a large number of tests is performed, compile statistic, raw p-value, and adjusted p-value for every feature into a single results table using tidyverse. The rationale for the non-parametric pair is that they are the direct alternatives provided by POMAShiny for the same two-/three-group structure; the rationale for the parametric pair is that POMAShiny exposes both parametric and non-parametric univariate methods side by side. Validate that the ST000284 three-group design was respected by the ANOVA/Kruskal-Wallis runs and that no feature lacks a statistic, p-value, or adjusted p-value.

## Related tools

- **POMAShiny** (Web-based front-end that hosts the univariate statistical analysis panel (t-test/ANOVA and Mann-Whitney/Kruskal-Wallis) with FDR correction) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the same univariate tests outside the web environment, on SummarizedExperiment objects) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (R packages used to compile test statistics, raw p-values, and adjusted p-values into one results table)
- **R** (Execution environment for the POMA/POMAShiny univariate testing workflow)

## Examples

```
library(POMA); library(tidyverse); se <- read.csv("features.csv") %>% PomaCreateSummExp(read.csv("target.csv")); res <- PomaUnivariate(se, method = "manova") # t-test/ANOVA + Mann-Whitney/Kruskal-Wallis with FDR
```

## Evaluation signals

- Every feature row in the output table has a non-empty statistic, raw p-value, and FDR-adjusted p-value
- The ST000284 three-group structure was respected — ANOVA/Kruskal-Wallis results reflect a three-group contrast, and t-test/Mann-Whitney results were not applied to that design
- Adjusted p-values are bounded between 0 and 1 and are greater than or equal to the corresponding raw p-values (FDR monotonicity)
- The number of output rows equals the number of features in the preprocessed feature matrix (one row per feature)
- Running the same feature table through POMAShiny's web panel and the POMA R package yields matching test statistics and adjusted p-values

## Limitations

- POMAShiny's univariate panel is limited to two-group (t-test, Mann-Whitney) and three-group (ANOVA, Kruskal-Wallis) contrasts; more complex designs are out of scope.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions due to substantial breaking changes in the API, so local deployment must pin the old version.
- No benchmarking or quantitative evaluation of POMAShiny against other univariate-testing tools is reported in the article.

## Evidence

- [discussion] FDR-adjusted p-values because a large number of tests is performed: [evidence span withheld — anchor status: not_found]
- [intro] univariate, multivariate and dimension reduction methods: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] statistical analysis panel within a four-panel workflow: "1) data upload, 2) preprocessing, 3) exploratory data analysis (EDA) and 4) statistical analysis"
- [results] requires two CSV files: target (metadata) and features: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] ST000284 targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
