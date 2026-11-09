---
name: nonparametric-rank-based-testing
description: Use when the target (metadata) design is a two- or three-group metabolomics/proteomics
  comparison whose features fail parametric assumptions, and per-feature p-values
  plus FDR adjustment are needed.
when_to_use_negative:
- The comparison is a two-group design where parametric t-test assumptions hold and
  you want POMAShiny's T-test/ANOVA panel instead — the article offers both, so pick
  the parametric panel when normality is defensible.
- Input is raw spectra needing spectral processing — the article directs such work
  to the RforMassSpectrometry packages first, and POMA is meant to be reached only
  after a feature matrix exists.
- Input data has not yet been through POMAShiny's imputation/normalization/outlier-removal
  pre-processing panels (e.g. features with >20% missing values per group are still
  present).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: Web interface providing the Mann-Whitney U / Kruskal-Wallis univariate analysis
    panel with FDR adjustment
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package that implements the rank-based univariate
    tests outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: Used for compiling and reshaping the per-feature results table
- name: R
  role: Execution environment for running POMA's univariate functions outside POMAShiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:70dad089d8417e8079c60dd3988088e766bb5221d0c44a84227136f25b5d1fb7
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Nonparametric rank-based testing

## Summary

Apply POMAShiny/POMA's rank-based univariate tests — Mann-Whitney U for two-group metabolomics/proteomics contrasts and Kruskal-Wallis for three-group designs — on the preprocessed feature matrix, with FDR-adjusted p-values computed for every tested feature.

## When to use

Use when the target (metadata) design is a two- or three-group metabolomics/proteomics comparison whose features fail parametric assumptions, and per-feature p-values plus FDR adjustment are needed.

## When NOT to use

- The comparison is a two-group design where parametric t-test assumptions hold and you want POMAShiny's T-test/ANOVA panel instead — the article offers both, so pick the parametric panel when normality is defensible.
- Input is raw spectra needing spectral processing — the article directs such work to the RforMassSpectrometry packages first, and POMA is meant to be reached only after a feature matrix exists.
- Input data has not yet been through POMAShiny's imputation/normalization/outlier-removal pre-processing panels (e.g. features with >20% missing values per group are still present).

## Inputs

- POMAShiny features file (CSV metabolite/peptide feature matrix, e.g. the ST000284 or ST000336 test data sets)
- POMAShiny target (metadata) CSV defining the two- or three-group design
- Preprocessed SummarizedExperiment/POMA data object after the imputation, normalization and outlier-removal panels

## Outputs

- Per-feature table of rank-based test statistics (Mann-Whitney U or Kruskal-Wallis)
- Raw p-values per feature
- FDR (multiple-testing) adjusted p-values per feature
- Ranked feature list for subsequent volcano/heatmap interpretation in the statistical analysis panel

## How to apply

Load the preprocessed POMAShiny feature matrix (already through the imputation/normalization panels) and its target CSV into a POMA/SummarizedExperiment object. Choose the test by group count: Mann-Whitney U for a two-group design (e.g. the ST000336-style contrast) and Kruskal-Wallis for a three-group design (e.g. ST000284); rank-based tests make no distributional assumption on each metabolite. Run the test per feature, collecting test statistics, raw p-values and p-values adjusted for multiple testing — the article notes that 'a large number of tests is performed in these analyses' because every feature is tested. Compile the results into one tidyverse-friendly table and inspect features passing the FDR threshold before interpreting individual metabolites. Validate the run by confirming the group structure in the target file drove the test choice (two groups -> Mann-Whitney, three groups -> Kruskal-Wallis) and that every feature row carries a non-empty statistic and both p-value columns.

## Related tools

- **POMAShiny** (Web interface providing the Mann-Whitney U / Kruskal-Wallis univariate analysis panel with FDR adjustment) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package that implements the rank-based univariate tests outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (Used for compiling and reshaping the per-feature results table)
- **R** (Execution environment for running POMA's univariate functions outside POMAShiny)

## Examples

```
R: se <- POMA::PomaSummarizedExperiment(features = read.csv("features.csv"), target = read.csv("target.csv")); res <- POMA::PomaUnivariate(se, method = "mann_whitney")
```

## Evaluation signals

- Output table has one row per feature with non-empty statistic, raw p-value and FDR-adjusted p-value columns.
- Test chosen matches the group count in the target file: Mann-Whitney U for 2 groups, Kruskal-Wallis for 3 groups (e.g. ST000284's three-group structure).
- Adjusted p-values differ from raw p-values and remain within [0, 1], indicating a multiple-testing correction was actually applied.
- No feature removed by the 20% missing-value cutoff still appears in the results (i.e. pre-processing preceded testing).
- Rank-based statistics are computed on the same feature set the parametric panel would test, allowing a side-by-side sanity check of the number of tested features.

## Limitations

- POMAShiny's univariate panel is limited to a fixed battery of tests (t-test/ANOVA, Mann-Whitney U/Kruskal-Wallis); more complex designs are outside its scope.
- The article does not report a benchmark of POMAShiny's univariate results against other tools, so correctness relies on the testthat-backed implementation and code coverage rather than external validation.
- Rank-based testing should be run only after the missing-value imputation, normalization and outlier-removal panels; raw input with >20% missing values per group will bias the tests.

## Evidence

- [results] POMAShiny's univariate analysis (T-test/ANOVA and Mann-Whitney U/Kruskal-Wallis) computes FDR-adjusted p-values, because a large number of tests is performed in these analyses.: [evidence span withheld — anchor status: not_found]
- [intro] including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [results] the example data set ST000336 consists of a targeted metabolomics two-group study: "the example data set ST000336 consists of a targeted metabolomics two-group study"
- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] POMAShiny workflow offers a missing value imputation panel composed of different operations divided into three sequential steps: "POMAShiny workflow offers a missing value imputation panel composed of different operations divided into three sequential steps"
- [results] it is highly recommended to perform an EDA before any statistical analysis: "it is highly recommended to perform an EDA before any statistical analysis"
