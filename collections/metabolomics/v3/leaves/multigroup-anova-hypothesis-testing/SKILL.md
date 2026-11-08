---
name: multigroup-anova-hypothesis-testing
description: Use when a numerical measurement column has more than two groups (e.g.
  a 'Condition' column) and the goal is a multi-group comparison — either ANOVA (parametric)
  or Kruskal-Wallis (non-parametric) — with the p-value annotated on the chart.
when_to_use_negative:
- The numerical column contains exactly two groups — use the t-test (parametric) or
  U-test (non-parametric Mann-Whitney) two-group activities instead.
- The activity cannot handle missing values and you need the raw source rows retained
  — the automatic pre-filtering step removes rows with missing values before submission,
  so raw data should not be required unchanged downstream.
- The goal is supervised prediction or feature selection (scikit-learn estimators
  with grid search), not group-comparison hypothesis testing.
edam_operation: http://edamontology.org/operation_0384
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: Instant Clue
  role: Interactive drag & drop platform used to dispatch ANOVA and Kruskal-Wallis
    activities onto generated charts and annotate resulting p-values
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Underlying runtime; the python source code is freely available so Instant
    Clue's statistical activities are editable and platform-independent
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:f973a90436f56b2ed8417512f199743d17bf59d3bba61f788f49a4dc8b77c59c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_0384
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multigroup-anova-hypothesis-testing

## Summary

Applies parametric (ANOVA) and non-parametric (Kruskal-Wallis) multi-group comparison tests inside Instant Clue's drag & drop Analysis toolbox, annotating the resulting p-value directly on the chart. The skill covers dispatching tests onto a base chart, automatic missing-value pre-filtering, and collecting per-column statistics for export.

## When to use

Use when a numerical measurement column has more than two groups (e.g. a 'Condition' column) and the goal is a multi-group comparison — either ANOVA (parametric) or Kruskal-Wallis (non-parametric) — with the p-value annotated on the chart.

## When NOT to use

- The numerical column contains exactly two groups — use the t-test (parametric) or U-test (non-parametric Mann-Whitney) two-group activities instead.
- The activity cannot handle missing values and you need the raw source rows retained — the automatic pre-filtering step removes rows with missing values before submission, so raw data should not be required unchanged downstream.
- The goal is supervised prediction or feature selection (scikit-learn estimators with grid search), not group-comparison hypothesis testing.

## Inputs

- Numerical measurement column (e.g. Body weight) from the loaded dataset
- Categorical grouping column (e.g. Condition) defining more than two groups
- Excel (.xlsx), tab-delimited (.txt), .csv, .xml, or compressed (.gz, .zip) data file

## Outputs

- Per-column multi-group test statistic and p-value (ANOVA or Kruskal-Wallis)
- P-value annotation rendered on the chart above lines connecting tested groups
- Exportable results table, e.g. statistical_tests_results.csv

## How to apply

Load the measurement data (e.g. Tutorial_Data_01.xlsx from Excel) into Instant Clue and drag the numerical column to the numerical data receiver box and the grouping column (e.g. Condition) to the categorical receiver box. Generate a base chart from these columns so statistical activities become available. For columns with more than two groups, drag the ANOVA activity from the Analysis toolbox for parametric testing, or the Kruskal-Wallis activity for non-parametric testing, onto the generated chart. Each test is an activity, so Instant Clue automatically filters out rows with missing values before submission without altering the source data. The computed p-value is annotated directly on the chart above the lines connecting the tested groups. Collect the per-column test statistics and p-values and export them, e.g. as statistical_tests_results.csv; judge correctness qualitatively by checking that every tested column yields a named test with a test statistic and p-value matching its group-comparison condition (multi-group; parametric vs. non-parametric).

## Related tools

- **Instant Clue** (Interactive drag & drop platform used to dispatch ANOVA and Kruskal-Wallis activities onto generated charts and annotate resulting p-values) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Underlying runtime; the python source code is freely available so Instant Clue's statistical activities are editable and platform-independent) — https://github.com/hnolCol/instantclue

## Examples

```
Drag ANOVA (or Kruskal-Wallis) activity from Analysis toolbox onto the chart built from 'Body weight' (numerical) + 'Condition' (categorical) in Instant Clue; export results as statistical_tests_results.csv
```

## Evaluation signals

- Every tested column yields a named test (ANOVA or Kruskal-Wallis) with a test statistic and a p-value, matching its group-comparison condition (more than two groups).
- The p-value annotation appears on the chart above the lines connecting the tested groups.
- Rows with missing values are automatically pre-filtered before the test activity while the source data remain unchanged.
- The test dispatch required a drag & drop action from the Analysis toolbox onto an already generated base chart.
- Results can be exported as a CSV (e.g. statistical_tests_results.csv) containing per-column statistics and p-values.

## Limitations

- The paper notes the authors are 'aiming to extend the panel of statistical tests' and will 'add further activities', so the current test suite is not exhaustive.
- No quantitative acceptance threshold is reported in the paper; correctness is judged qualitatively rather than against a numeric cutoff.
- Instant Clue is a Python >=3.4 desktop application (PyQt6 since 0.12.0); running the tests outside the GUI requires the source-code route (python3 -m venv env; pip install -r requirements.txt).
- Automatic pre-filtering changes the effective sample per test when missing values are present, so group sizes may differ from the raw dataset.

## Evidence

- [results] ANOVA and Kruskal-Wallis dispatch rule: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] Automatic pre-filtering of missing values: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Defining groups by drag & drop: "adding the Body weight column to the numerical data receiver box and the Condition column to the categorical receiver box by drag & drop"
- [results] Chart-driven p-value annotation: [evidence span withheld — anchor status: not_found]
- [results] Input data format support: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Runtime requirement: "create a virtual environment to not mix up required package versions with your python installation using 
```
python3 -m venv env #mac 
source env/bin/activate # activate env
```"
