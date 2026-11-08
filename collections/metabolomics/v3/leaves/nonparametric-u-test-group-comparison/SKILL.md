---
name: nonparametric-u-test-group-comparison
description: Use when a numerical column has exactly two categorical groups (e.g.
  Condition), the data may violate parametric assumptions, and an interactive p-value
  for the two-group comparison is needed on a chart.
when_to_use_negative:
- The grouping column contains more than two groups — use ANOVA or Kruskal-Wallis
  multi-group activities instead
- The data are known to satisfy parametric assumptions and a t-test is preferred for
  the two-group comparison
- The goal is batch-effect correction or prediction modeling, which are separate Instant
  Clue activities (e.g. pyComBat combat, scikit-learn estimators), not two-group U-testing
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: GUI application in which the U-test activity is dragged from the Analysis
    toolbox onto a generated chart to run the two-group non-parametric comparison
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for Instant Clue's source code, making the statistical test implementation
    editable and platform-independent
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:1299547fbc281449e6d2741597c765f9a407005f03542b06a1cafb9fe49c989d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# nonparametric-u-test-group-comparison

## Summary

Performs two-group non-parametric comparison (Mann-Whitney U-test) in Instant Clue by dragging the U-test activity from the Analysis toolbox onto a generated chart, yielding a per-column test statistic and p-value annotated directly on the chart.

## When to use

Use when a numerical column has exactly two categorical groups (e.g. Condition), the data may violate parametric assumptions, and an interactive p-value for the two-group comparison is needed on a chart.

## When NOT to use

- The grouping column contains more than two groups — use ANOVA or Kruskal-Wallis multi-group activities instead
- The data are known to satisfy parametric assumptions and a t-test is preferred for the two-group comparison
- The goal is batch-effect correction or prediction modeling, which are separate Instant Clue activities (e.g. pyComBat combat, scikit-learn estimators), not two-group U-testing

## Inputs

- Excel file (e.g. Tutorial_Data_01.xlsx) with a numerical measurement column and a categorical grouping column with exactly two groups
- A base chart generated in Instant Clue from the numerical and categorical receiver boxes

## Outputs

- Per-column U-test statistic and p-value annotated on the chart above the connecting lines between groups
- Exported table of per-column test statistics and p-values (statistical_tests_results.csv)

## How to apply

Load the numerical data (e.g. from Tutorial_Data_01.xlsx, an Excel file supported by Instant Clue) and drag the measurement column into the numerical data receiver box and the grouping column into the categorical receiver box to define the comparison groups. Generate a base chart from these columns, then dispatch the U-test (non-parametric Mann-Whitney) activity by dragging it from the Analysis toolbox onto the chart — two-group tests are enabled only after this drag & drop action. Before each test, Instant Clue automatically pre-filters rows with missing values, since an activity that cannot handle missing values has the data filtered before submission without changing the source data. The resulting p-value is annotated directly on the chart above the lines connecting the tested groups; collect per-column statistics and p-values (e.g. export as statistical_tests_results.csv) and validate qualitatively that each tested column yields a named test with a statistic and p-value.

## Related tools

- **Instant Clue** (GUI application in which the U-test activity is dragged from the Analysis toolbox onto a generated chart to run the two-group non-parametric comparison) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for Instant Clue's source code, making the statistical test implementation editable and platform-independent) — https://github.com/hnolCol/instantclue

## Evaluation signals

- Every tested column yields a named test (U-test) with a test statistic and p-value matching its two-group, non-parametric condition
- The p-value appears annotated on the chart above the lines connecting the tested groups
- Rows with missing values were removed by automatic pre-filtering before test submission, without altering the source data
- The comparison was only calculable after the drag & drop dispatch of the U-test activity onto the chart
- Per-column results export cleanly as statistical_tests_results.csv

## Limitations

- No quantitative acceptance threshold for the test result is reported; validation is qualitative (named test, statistic, p-value present)
- The U-test applies only to columns with exactly two groups; multi-group data require ANOVA or Kruskal-Wallis
- Instant Clue's panel of statistical tests is still being extended — additional activities for systematic data interpretation are planned, not yet implemented

## Evidence

- [results] Two-group dispatch of U-test via drag & drop: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] Automatic missing-value pre-filtering before activities: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Chart generation via receiver boxes: "adding the Body weight column to the numerical data receiver box and the Condition column to the categorical receiver box by drag & drop"
- [results] Supported input file formats including Excel: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [intro] Python runtime availability: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
