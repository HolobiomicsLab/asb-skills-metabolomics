---
name: polynomial-regression-fitting
description: Use when numeric rows (e.g. Tutorial_Data_01.xlsx expression or enzyme-kinetics
  columns) must each be fitted with a linear, polynomial, or Michaelis-Menten model
  and compared to observations via Pearson correlation.
when_to_use_negative:
- The goal is unsupervised exploration (dimensional reduction, (un)-supervised learning,
  clustering, multi-block omics integration) rather than fitting a parametric curve
  per row
- The data are categorical annotations or labels rather than numeric response/concentration-series
  values suited to linear, polynomial, or Michaelis-Menten models
- The rows are time-course expression expected to be circadian/periodic — use the
  toolbox's periodic-function fits instead of polynomial regression, since a polynomial
  will not capture rhythmic behavior
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: Provides the curve fitting toolbox whose fit functions are applied row-wise
    in a customizable way (linear, polynomial, Michaelis-Menten, periodic) alongside
    correlation-based analysis
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: 'Runtime for replicating the workflow: read the .xlsx input with pandas, pre-filter
    missing rows, run least-squares/nonlinear least-squares fits, and write curve_fit_results.csv'
- name: scikit-learn
  role: Underlying library Instant Clue uses to establish estimators for prediction,
    covering the regression-style fitting step of this skill
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:7502202cabe20a813deb1dc4d62c7a9f73846e7488d59440f0042c230a048726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# polynomial-regression-fitting

## Summary

Row-wise least-squares curve fitting (linear, polynomial, Michaelis-Menten) as implemented in Instant Clue's curve fitting toolbox, used to connect phenotype-characterizing data to proteomic or genomic expression data, with Pearson correlation of fitted curves to observed values.

## When to use

Use when numeric rows (e.g. Tutorial_Data_01.xlsx expression or enzyme-kinetics columns) must each be fitted with a linear, polynomial, or Michaelis-Menten model and compared to observations via Pearson correlation.

## When NOT to use

- The goal is unsupervised exploration (dimensional reduction, (un)-supervised learning, clustering, multi-block omics integration) rather than fitting a parametric curve per row
- The data are categorical annotations or labels rather than numeric response/concentration-series values suited to linear, polynomial, or Michaelis-Menten models
- The rows are time-course expression expected to be circadian/periodic — use the toolbox's periodic-function fits instead of polynomial regression, since a polynomial will not capture rhythmic behavior

## Inputs

- Spreadsheet of numeric per-row measurements (e.g. Tutorial_Data_01.xlsx), readable via Instant Clue's Excel/.xlsx, .txt, .csv, .xml, .gz, or .zip upload path
- Selected target numeric columns for fitting (with rows containing missing values in those columns removed beforehand)

## Outputs

- curve_fit_results.csv containing fitted parameter rows for the linear, polynomial, and Michaelis-Menten models
- A correlation-coefficient field (Pearson r) for each fit, joining fitted curves back to the observed data

## How to apply

Load the spreadsheet (e.g. Tutorial_Data_01.xlsx) via the Excel-reading path supported by Instant Clue into a pandas DataFrame, then pre-filter rows with missing values in the target columns, mirroring how Instant Clue automatically filters data before activities that cannot handle missing values. Fit a linear model and a polynomial model by least-squares regression to estimate the coefficients, and fit the enzymatic Michaelis-Menten model by nonlinear least-squares to estimate Vmax and Km. For each model, compute Pearson correlation coefficients between the fitted curve and the observed data on the same row. Instant Clue's toolbox applies these fit functions row-wise in a customizable way, including periodic functions for identifying genes/proteins following circadian rhythm; restrict model choice to the models justified by the data shape. Assemble all fitted parameters and correlation coefficients into a single table written to curve_fit_results.csv, keeping one parameter row per model per fitted row.

## Related tools

- **Instant Clue** (Provides the curve fitting toolbox whose fit functions are applied row-wise in a customizable way (linear, polynomial, Michaelis-Menten, periodic) alongside correlation-based analysis) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for replicating the workflow: read the .xlsx input with pandas, pre-filter missing rows, run least-squares/nonlinear least-squares fits, and write curve_fit_results.csv)
- **scikit-learn** (Underlying library Instant Clue uses to establish estimators for prediction, covering the regression-style fitting step of this skill)

## Examples

```
import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx').dropna(subset=target_cols)  # then fit linear/polynomial by least squares and Michaelis-Menten by nonlinear least squares, write curve_fit_results.csv
```

## Evaluation signals

- curve_fit_results.csv exists and contains fitted parameter rows for all three models: linear, polynomial, and Michaelis-Menten
- Each fit row carries a Pearson correlation-coefficient field computed against the observed data
- The Michaelis-Menten rows report estimated kinetic parameters Vmax and Km
- Rows with missing values in the target columns were excluded before fitting, matching Instant Clue's automatic pre-filtering behavior
- All fitted parameters and correlation coefficients are assembled into a single table (one merged output, not per-model files)

## Limitations

- Activities such as curve fitting cannot handle missing values, so rows are automatically pre-filtered before the activity without changing the source data — rows lost to missingness reduce the fitted set
- The executable is distributed for Mac OS and Windows; building from source requires Python >=3.4 and package-version care (pyComBat must be installed first because of specific pandas/numpy versioning requirements)
- A polynomial fit is not appropriate for periodic phenomena such as circadian-rhythm genes/proteins, which the toolbox addresses with dedicated periodic functions

## Evidence

- [abstract] row-wise curve fitting support in Instant Clue: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting"
- [intro] regressions and curve fitting among the statistical repertoire: "(iii) Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis as well as facilitating calculation of metrics like area under the curve (AUC)"
- [results] automatic pre-filtering of rows with missing values before fitting activities: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] row-wise customizable fit functions including enzymatic models: [evidence span withheld — anchor status: not_found]
- [readme] Python-based desktop implementation and execution entry point: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
