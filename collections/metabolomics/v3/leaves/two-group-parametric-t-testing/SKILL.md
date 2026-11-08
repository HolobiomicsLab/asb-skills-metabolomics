---
name: two-group-parametric-t-testing
description: Use when a numerical measurement column must be compared across exactly
  two groups defined by a categorical column (e.g. Condition) in Instant Clue, and
  a parametric t-test is appropriate for the data distribution.
when_to_use_negative:
- The grouping column has more than two groups — use the ANOVA (parametric) or Kruskal-Wallis
  (non-parametric) activity instead
- The parametric assumption is not justified for the data — dispatch the U-test (non-parametric
  Mann-Whitney) activity instead of the t-test
- The data has no categorical grouping column to assign to the categorical receiver
  box, so no two-group comparison can be defined via drag & drop
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Instant Clue
  role: GUI desktop application in which the t-test activity is dispatched via drag
    & drop from the Analysis toolbox onto the generated chart
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Underlying open-source runtime; the Python source code of Instant Clue is
    freely available, making the test implementation inspectable and platform-independent
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:491ec26759dccfbeccd461761c7c36afdae1a0026c500a0b7002b5bdf4aaf409
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# two-group-parametric-t-testing

## Summary

Dispatches a two-group parametric t-test inside Instant Clue by dragging the t-test activity from the Analysis toolbox onto a chart built from a numerical column and a categorical grouping column (e.g. Body weight vs Condition); the resulting p-value is annotated directly on the chart above the line connecting the tested groups.

## When to use

Use when a numerical measurement column must be compared across exactly two groups defined by a categorical column (e.g. Condition) in Instant Clue, and a parametric t-test is appropriate for the data distribution.

## When NOT to use

- The grouping column has more than two groups — use the ANOVA (parametric) or Kruskal-Wallis (non-parametric) activity instead
- The parametric assumption is not justified for the data — dispatch the U-test (non-parametric Mann-Whitney) activity instead of the t-test
- The data has no categorical grouping column to assign to the categorical receiver box, so no two-group comparison can be defined via drag & drop

## Inputs

- Numerical measurement column (e.g. Body weight) from a loaded dataset (Excel, .txt, .csv, .xml, .gz, .zip)
- Categorical grouping column with exactly two groups (e.g. Condition)
- A base chart generated from the numerical + categorical column pair

## Outputs

- p-value annotated on the chart above the line connecting the two tested groups
- Per-column test statistics and p-values exportable as statistical_tests_results.csv

## How to apply

Load the data (e.g. Tutorial_Data_01.xlsx, which supports Excel upload) and drag the numerical measurement column into the numerical data receiver box and the grouping column into the categorical receiver box by drag & drop, then generate a base chart from these columns. Drag the t-test (parametric) activity from the Analysis toolbox onto the generated chart; this interactive dispatch triggers automatic calculation and annotation of the p-value on the chart above the lines connecting the tested groups. Before submission, Instant Clue automatically pre-filters rows with missing values, since the test activity cannot handle NaNs, without altering the source data. Collect the per-column test statistics and p-values and export them (e.g. as statistical_tests_results.csv); choose the U-test (Mann-Whitney, non-parametric) activity instead of the t-test when the parametric assumption is not justified, and ANOVA/Kruskal-Wallis when more than two groups are present.

## Related tools

- **Instant Clue** (GUI desktop application in which the t-test activity is dispatched via drag & drop from the Analysis toolbox onto the generated chart) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Underlying open-source runtime; the Python source code of Instant Clue is freely available, making the test implementation inspectable and platform-independent) — https://github.com/hnolCol/instantclue

## Evaluation signals

- Every tested two-group column yields a named test (t-test) with a test statistic and a p-value matching the two-group, parametric condition
- The resulting p-value is annotated directly on the chart above the lines connecting the tested groups
- Rows with missing values were removed before each test activity without changing the source data (automatic pre-filtering)
- The t-test activity was only dispatchable after a drag & drop action onto a chart built from a numerical and a categorical column
- Exported statistical_tests_results.csv contains per-column test statistics and p-values for all tested columns

## Limitations

- No quantitative acceptance threshold for the test results is reported in the paper; validation is qualitative (each column yields a named test with statistic and p-value)
- Missing values cannot be handled by the test activity itself — data are automatically filtered, which removes affected rows from the analysis
- The panel of statistical tests is still being extended; further activities for systematic data interpretation were planned but not yet added at publication time

## Evidence

- [results] Interactive dispatch of two-group tests: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] Automatic pre-filtering before each test activity: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Defining comparison groups via drag & drop: "adding the Body weight column to the numerical data receiver box and the Condition column to the categorical receiver box by drag & drop"
- [readme] Underlying technology of the GUI application: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
- [results] Supported upload formats for input data: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv)"
