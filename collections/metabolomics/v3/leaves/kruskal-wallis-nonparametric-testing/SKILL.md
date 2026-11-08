---
name: kruskal-wallis-nonparametric-testing
description: Use when a loaded dataset column (e.g. from Tutorial_Data_01.xlsx) has
  more than two groups defined by a categorical Condition column and a non-parametric
  multi-group comparison of the numerical measurement is required.
when_to_use_negative:
- The tested column has exactly two groups — use the t-test (parametric) or U-test
  (Mann-Whitney, non-parametric) two-group activity instead
- The data are roughly normally distributed and a parametric multi-group test (ANOVA)
  is appropriate
- A statistical panel beyond Instant Clue's current activities is needed — the paper
  states the panel of statistical tests is a planned extension, so heavier custom
  testing may require the Python source directly
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI (Python/PyQt6) in which the Kruskal-Wallis activity is dispatched
    via drag & drop from the Analysis toolbox onto generated charts
  repo: https://github.com/hnolCol/instantclue
- name: Python
  role: Underlying runtime for Instant Clue (>=3.4); the python source code is freely
    available making Instant Clue editable on any platform
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:7b602e16382ee4bf565c3c3b5986b19807dcbc69c7d59444f7bc7efc823a40ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kruskal-wallis-nonparametric-testing

## Summary

Perform the Kruskal-Wallis non-parametric multi-group comparison in Instant Clue by dragging the Kruskal-Wallis activity from the Analysis toolbox onto a generated chart of a column with more than two groups, with automatic pre-filtering of missing values and p-value annotation on the chart.

## When to use

Use when a loaded dataset column (e.g. from Tutorial_Data_01.xlsx) has more than two groups defined by a categorical Condition column and a non-parametric multi-group comparison of the numerical measurement is required.

## When NOT to use

- The tested column has exactly two groups — use the t-test (parametric) or U-test (Mann-Whitney, non-parametric) two-group activity instead
- The data are roughly normally distributed and a parametric multi-group test (ANOVA) is appropriate
- A statistical panel beyond Instant Clue's current activities is needed — the paper states the panel of statistical tests is a planned extension, so heavier custom testing may require the Python source directly

## Inputs

- Tutorial_Data_01.xlsx (Excel file upload supported; also .txt, .csv, .xml, .gz, .zip)
- A numerical measurement column (e.g. Body weight) assigned to the numerical data receiver box
- A categorical grouping column (e.g. Condition) with more than two groups assigned to the categorical receiver box
- A base chart generated from these columns

## Outputs

- Kruskal-Wallis p-value annotated on the chart above the lines connecting the tested groups
- statistical_tests_results.csv with per-column test statistics and p-values

## How to apply

Load the data (e.g. Tutorial_Data_01.xlsx) into Instant Clue, assign the numerical measurement column to the numerical data receiver box and the grouping column (e.g. Condition) to the categorical receiver box by drag & drop, and generate a base chart from these columns. For columns with more than two groups, drag the Kruskal-Wallis (non-parametric) activity from the Analysis toolbox onto the generated chart; each test is an activity in Instant Clue's activity model. Because Kruskal-Wallis cannot handle missing values, Instant Clue automatically filters out rows with missing values before submission to the activity, without changing the source data. The calculated p-value is annotated directly on the chart above the lines connecting the tested groups, and per-column test statistics and p-values can be collected and exported as statistical_tests_results.csv.

## Related tools

- **Instant Clue** (Desktop GUI (Python/PyQt6) in which the Kruskal-Wallis activity is dispatched via drag & drop from the Analysis toolbox onto generated charts) — https://github.com/hnolCol/instantclue
- **Python** (Underlying runtime for Instant Clue (>=3.4); the python source code is freely available making Instant Clue editable on any platform) — https://github.com/hnolCol/instantclue

## Evaluation signals

- Every tested column yields a named test (Kruskal-Wallis for >2 groups, not ANOVA) with a test statistic and p-value
- The dispatched test matches its group-comparison condition: multi-group (Kruskal-Wallis) vs. two-group (U-test) and non-parametric vs. parametric
- The p-value appears annotated on the chart above the lines connecting the tested groups
- Source data are unchanged after the test despite the automatic pre-filtering of rows with missing values
- Exported statistical_tests_results.csv contains per-column entries with a test statistic and p-value

## Limitations

- The assessment/test is enabled only after a drag & drop action; there is no batch or scripted dispatch within the GUI
- The paper reports no quantitative acceptance threshold, so validation of results is qualitative
- The panel of statistical tests is not yet fully implemented — the authors state they aim to extend the panel of statistical tests and add further activities

## Evidence

- [results] activity dispatch: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] missing value pre-filtering: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] p-value reporting: [evidence span withheld — anchor status: not_found]
- [results] interactive test triggering: [evidence span withheld — anchor status: not_found]
- [readme] availability: "Please find the executables for Windows and Mac in the [Releases](https://github.com/hnolCol/instantclue/releases) section."
