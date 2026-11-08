---
name: curve-fitting-parameter-estimation
description: Use when connecting phenotype-characterizing data (e.g. immunoblot densitometry,
  qPCR, time-series expression from Tutorial_Data_01.xlsx) to expression trends via
  linear, polynomial or Michaelis-Menten fits with per-row correlation.
when_to_use_negative:
- The activity cannot handle missing values and data have not been pre-filtered —
  Instant Clue automatically filters before such activities, so feeding raw rows with
  NaNs is wrong.
- The goal is multi-block omics integration, dimensional reduction, or clustering
  — use Instant Clue's other statistical activities rather than the curve fitting
  toolbox.
- The data are batch-affected omics matrices needing ComBat correction — pyComBat
  is the dedicated (separate) tool in this project's stack.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Instant Clue
  role: Provides the curve fitting toolbox with row-wise, customizable fits (linear,
    polynomial, Michaelis-Menten, periodic) plus correlation-based analysis over .xlsx
    inputs
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runs the fit pipeline (pandas loading, least-squares fitting, writing curve_fit_results.csv);
    Instant Clue's source is editable Python
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Underlying library Instant Clue uses to establish estimators for prediction,
    complementing the regression/fit activities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:e3f96defe6412b4b8a5ecd72c11db4d33c22ec1b523eac8be6c7a7c56825ae1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# curve-fitting-parameter-estimation

## Summary

Row-wise curve fitting and correlation analysis over tabular numeric data, as implemented in Instant Clue's curve fitting toolbox: least-squares linear/polynomial fits, nonlinear least-squares Michaelis-Menten kinetic fitting, and Pearson correlation of fitted curves against observations.

## When to use

Use when connecting phenotype-characterizing data (e.g. immunoblot densitometry, qPCR, time-series expression from Tutorial_Data_01.xlsx) to expression trends via linear, polynomial or Michaelis-Menten fits with per-row correlation.

## When NOT to use

- The activity cannot handle missing values and data have not been pre-filtered — Instant Clue automatically filters before such activities, so feeding raw rows with NaNs is wrong.
- The goal is multi-block omics integration, dimensional reduction, or clustering — use Instant Clue's other statistical activities rather than the curve fitting toolbox.
- The data are batch-affected omics matrices needing ComBat correction — pyComBat is the dedicated (separate) tool in this project's stack.

## Inputs

- Tutorial_Data_01.xlsx spreadsheet (or .txt/.csv/.xml/.gz/.zip equivalents) loaded into a pandas DataFrame
- Selected numerical columns of phenotype-characterizing and expression data (rows = proteins/genes/conditions)

## Outputs

- curve_fit_results.csv containing fitted parameter rows for linear, polynomial, and Michaelis-Menten (Vmax, Km) models
- A Pearson correlation-coefficient field per fit relating fitted curves to observed data

## How to apply

Load the spreadsheet (Instant Clue supports .xlsx, .txt, .csv, .xml, .gz, .zip) into a pandas DataFrame; Tutorial_Data_01.xlsx is the paper's bundled example. Pre-filter rows with missing values in target columns before fitting, mirroring Instant Clue's automatic pre-filtering of data before activities that cannot handle missing values. Fit linear and polynomial models by least-squares regression to estimate model coefficients row-wise, then fit the Michaelis-Menten enzymatic model by nonlinear least-squares to estimate Vmax and Km. Compute Pearson correlation coefficients between each fitted curve and the observed data. Assemble all fitted parameters and correlation coefficients into a single table written to curve_fit_results.csv. Instant Clue also provides periodic functions for identifying genes/proteins following circadian rhythm, so a periodic model can be substituted when temporal data suggest rhythmicity.

## Related tools

- **Instant Clue** (Provides the curve fitting toolbox with row-wise, customizable fits (linear, polynomial, Michaelis-Menten, periodic) plus correlation-based analysis over .xlsx inputs) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runs the fit pipeline (pandas loading, least-squares fitting, writing curve_fit_results.csv); Instant Clue's source is editable Python) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Underlying library Instant Clue uses to establish estimators for prediction, complementing the regression/fit activities)

## Examples

```
df = pd.read_excel('Tutorial_Data_01.xlsx').dropna(subset=cols); from scipy.optimize import curve_fit; (Vmax, Km), _ = curve_fit(michaelis_menten, x, y)
```

## Evaluation signals

- curve_fit_results.csv exists and is non-empty after the run
- The output table contains fitted parameter rows for all three models: linear, polynomial, and Michaelis-Menten
- Michaelis-Menten rows expose both kinetic parameters Vmax and Km
- Each fit has an associated Pearson correlation-coefficient field
- Rows with missing values in the target columns were excluded before fitting (consistent with the pre-filtering behaviour)

## Limitations

- Activities that cannot handle missing values require pre-filtering; Instant Clue filters automatically without changing source data, and a custom pipeline must replicate this
- Curve fitting is restricted to the toolbox's built-in fit families (linear, polynomial, Michaelis-Menten, periodic/circadian); arbitrary user-defined models are not covered
- The paper notes Instant Clue's statistical-test panel is planned for extension, so some statistical activities may be absent depending on version

## Evidence

- [abstract] it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting"
- [intro] Objective (iii): regressions and curve fitting: "Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis as well as facilitating calculation of metrics like area under the curve (AUC)"
- [results] Automatic pre-filtering before activities that cannot handle missing values: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] scikit-learn based estimators: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [intro] Python source availability: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
