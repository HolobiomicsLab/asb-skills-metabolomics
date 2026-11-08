---
name: correlation-coefficient-computation
description: Use when you need to quantify agreement between fitted curves (linear,
  polynomial, Michaelis-Menten, periodic) and observed expression or phenotype measurements,
  e.g. scoring per-row fit quality from least-squares models on spreadsheet data.
when_to_use_negative:
- The activity you are about to run cannot handle missing values and the data have
  not been pre-filtered — Instant Clue filters such rows automatically before the
  activity, and skipping this makes correlations ill-defined
- You need model-free dissimilarity or clustering of candidates rather than agreement
  between a parametric fit and observed values (use Instant Clue's hierarchical clustering
  or selection tool instead)
- The goal is prediction/estimation rather than correlation of fitted vs observed
  curves — use Instant Clue's scikit-learn-based estimator pipeline with grid search
  instead
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: Provides the curve fitting and correlation toolbox that applies fits (linear,
    polynomial, Michaelis-Menten, periodic) row-wise and supports correlation-based
    analysis between fitted and observed data
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for Instant Clue's open-source codebase; used to load Tutorial_Data_01.xlsx
    into a pandas DataFrame, run the fits, and write curve_fit_results.csv
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Underlying library for Instant Clue's estimator/prediction functions that
    can accompany fit-based analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:1d9518ca586806cb8f713087afaea04a34cce809c2f71a7bdb7c8b2a9d83fe64
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
triage_status: promoted
yes_rate: 0.6857142857142857
n_claims: 35
---

# correlation-coefficient-computation

## Summary

Computes Pearson correlation coefficients linking fitted models (linear, polynomial, Michaelis-Menten, circadian/periodic) to observed phenotype-characterizing or omics expression data, as implemented row-wise in Instant Clue's curve fitting and correlation toolbox.

## When to use

Use when you need to quantify agreement between fitted curves (linear, polynomial, Michaelis-Menten, periodic) and observed expression or phenotype measurements, e.g. scoring per-row fit quality from least-squares models on spreadsheet data.

## When NOT to use

- The activity you are about to run cannot handle missing values and the data have not been pre-filtered — Instant Clue filters such rows automatically before the activity, and skipping this makes correlations ill-defined
- You need model-free dissimilarity or clustering of candidates rather than agreement between a parametric fit and observed values (use Instant Clue's hierarchical clustering or selection tool instead)
- The goal is prediction/estimation rather than correlation of fitted vs observed curves — use Instant Clue's scikit-learn-based estimator pipeline with grid search instead

## Inputs

- Numerical expression/phenotype matrix loaded from an Excel (.xlsx), .txt, .csv, .xml, or compressed (.gz, .zip) file into a pandas DataFrame
- Fitted model curves from linear, polynomial, Michaelis-Menten, or periodic fits on the same rows

## Outputs

- curve_fit_results.csv containing fitted parameter rows for linear, polynomial, and Michaelis-Menten models with a correlation-coefficient field for each fit
- Per-row Pearson correlation coefficients between fitted curves and observed data

## How to apply

Load the numerical data (e.g. Tutorial_Data_01.xlsx via Instant Clue's Excel-reading path, which supports Excel, .txt, .csv, .xml, .gz and .zip inputs) into a pandas DataFrame and pre-filter rows with missing values in the target columns, mirroring Instant Clue's automatic pre-filtering before activities that cannot handle missing values. Fit each candidate model — linear and polynomial by least-squares regression, Michaelis-Menten by nonlinear least-squares to estimate Vmax and Km, and periodic functions for circadian-rhythm genes/proteins — applied row-wise in a customizable way. Then compute Pearson correlation coefficients between each fitted curve and the observed data per model. Assemble the fitted parameters and correlation coefficients into a single table (curve_fit_results.csv) so model choice can be judged by the correlation field alongside parameter estimates.

## Related tools

- **Instant Clue** (Provides the curve fitting and correlation toolbox that applies fits (linear, polynomial, Michaelis-Menten, periodic) row-wise and supports correlation-based analysis between fitted and observed data) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for Instant Clue's open-source codebase; used to load Tutorial_Data_01.xlsx into a pandas DataFrame, run the fits, and write curve_fit_results.csv) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Underlying library for Instant Clue's estimator/prediction functions that can accompany fit-based analysis)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(y_obs, y_fit_mm)
```

## Evaluation signals

- curve_fit_results.csv exists and contains separate fitted parameter rows for the linear, polynomial, and Michaelis-Menten models
- Each fit row includes a correlation-coefficient field; Michaelis-Menten rows report both Vmax and Km estimates
- Correlation coefficients fall within the valid [-1, 1] range for every fitted row
- Rows with missing values in the target columns were pre-filtered before fitting, without altering the source data
- Pearson correlations of the fitted curves against observed data are consistent with residuals from the least-squares fits

## Limitations

- Activities that cannot handle missing values require automatic or manual pre-filtering first; the source data are not changed, so downstream results depend on that filtering
- The panel of statistical tests and further activities for systematic data interpretation is still being extended in Instant Clue, so some test types may not be available
- Periodic (circadian) fits are model-dependent; a poor Pearson correlation may reflect an inappropriate model choice rather than no relationship

## Evidence

- [abstract] Portfolio of statistical tools including curve fitting: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting"
- [intro] Objective (iii) naming regressions and curve fitting: "Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis"
- [results] Automatic pre-filtering before activities that cannot handle missing values: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Supported input file types for loading data: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Instant Clue is Python-based desktop application: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
