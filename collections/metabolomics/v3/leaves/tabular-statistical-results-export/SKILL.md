---
name: tabular-statistical-results-export
description: Use when a tabular dataset (e.g. Excel/csv with a numerical measurement
  column and a categorical Condition column) needs two-group or multi-group parametric/non-parametric
  comparisons, and the named test, statistic and p-value per column must be collected
  into one machine-readable CSV.
when_to_use_negative:
- The data are not tabular group-comparison data with a numerical measurement plus
  a categorical group column (e.g. raw omics peak lists or image data)
- Only one group is present per comparison — the t-test/U-test activities are enabled
  only for columns with exactly two groups and ANOVA/Kruskal-Wallis for more than
  two groups
- The goal is batch-effect correction or prediction modelling (e.g. ComBat via pyComBat
  or scikit-learn pipelines) rather than group-comparison testing
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI in which the t-test, U-test, ANOVA and Kruskal-Wallis activities
    are dispatched onto charts by drag & drop and the p-value/statistic results are
    collected and exported
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for the Instant Clue source, usable to drive the per-column test dispatch
    and write the statistical_tests_results.csv export
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:82cde7e2d42580f41cc2b63e4e75f35941d0284b828c58354a86bf07573f7cb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tabular-statistical-results-export

## Summary

Dispatch interactive group-comparison statistical tests (t-test, U-test, ANOVA, Kruskal-Wallis) in Instant Clue on a loaded dataset such as Tutorial_Data_01.xlsx and export the per-column test statistics and p-values as a flat statistical_tests_results.csv table for reuse.

## When to use

Use when a tabular dataset (e.g. Excel/csv with a numerical measurement column and a categorical Condition column) needs two-group or multi-group parametric/non-parametric comparisons, and the named test, statistic and p-value per column must be collected into one machine-readable CSV.

## When NOT to use

- The data are not tabular group-comparison data with a numerical measurement plus a categorical group column (e.g. raw omics peak lists or image data)
- Only one group is present per comparison — the t-test/U-test activities are enabled only for columns with exactly two groups and ANOVA/Kruskal-Wallis for more than two groups
- The goal is batch-effect correction or prediction modelling (e.g. ComBat via pyComBat or scikit-learn pipelines) rather than group-comparison testing

## Inputs

- Tabular data file such as Tutorial_Data_01.xlsx (Excel, .txt, .csv, .xml, .gz or .zip uploads supported)
- Numerical measurement column assigned to the numerical data receiver box
- Categorical grouping column (e.g. Condition) assigned to the categorical receiver box
- A base chart generated from those columns onto which test activities are dropped

## Outputs

- statistical_tests_results.csv with per-column named test, test statistic and p-value
- Chart annotations showing each pairwise p-value above the lines connecting the tested groups

## How to apply

Load the data into Instant Clue (Excel, txt, csv, xml, gz or zip are accepted upload formats), e.g. the accompanied Tutorial_Data_01.xlsx. Assign the numerical measurement column to the numerical data receiver box and the grouping column (e.g. Condition) to the categorical receiver box by drag & drop, then generate a base chart from these columns so statistical activities can be applied. Dispatch the test as an activity: drag the t-test (parametric) or U-test (Mann-Whitney, non-parametric) onto the chart for columns with exactly two groups, and ANOVA (parametric) or Kruskal-Wallis (non-parametric) for columns with more than two groups. Each test is an activity, so rows with missing values are automatically pre-filtered before submission without altering the source data. Collect the per-column test statistics and p-values, noting that Instant Clue also annotates the resulting p-value directly on the chart above the lines connecting the tested groups, and export the collected results as statistical_tests_results.csv (optionally via the bundled Python source, which requires Python >=3.4).

## Related tools

- **Instant Clue** (Desktop GUI in which the t-test, U-test, ANOVA and Kruskal-Wallis activities are dispatched onto charts by drag & drop and the p-value/statistic results are collected and exported) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for the Instant Clue source, usable to drive the per-column test dispatch and write the statistical_tests_results.csv export) — https://github.com/hnolCol/instantclue

## Examples

```
df[['column','test','statistic','p_value']].to_csv('statistical_tests_results.csv', index=False)
```

## Evaluation signals

- Every tested column in statistical_tests_results.csv yields a named test (t-test, U-test, ANOVA or Kruskal-Wallis) plus a test statistic and p-value
- The named test matches the group-comparison condition: t-test/U-test only for columns with exactly two groups, ANOVA/Kruskal-Wallis only for columns with more than two groups
- Parametric (t-test, ANOVA) and non-parametric (U-test, Kruskal-Wallis) variants are each selectable so results can be cross-checked against one another per column
- Source data are unchanged: missing-value rows are pre-filtered per activity without altering the source data
- Annotated p-values on the chart agree with the p-values exported in the CSV

## Limitations

- The paper reports no quantitative acceptance threshold for the exported results, so correctness is judged qualitatively (named test + statistic + p-value per column)
- The authors state they are aiming to extend the panel of statistical tests and add further activities, so the test repertoire may be incomplete
- Two-group test activities are enabled only after a drag & drop action onto a chart, so tests cannot be dispatched without first generating a base chart

## Evidence

- [results] statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] automatic missing-value pre-filtering before each test activity: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] chart column assignment via drag & drop: "adding the Body weight column to the numerical data receiver box and the Condition column to the categorical receiver box by drag & drop"
- [intro] repertoire of statistical tests offered: "(iii) Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis"
- [results] bundled tutorial dataset: [evidence span withheld — anchor status: not_found]
