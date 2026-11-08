---
name: missing-value-prefiltering-before-statistical-activities
description: Use when dispatching a statistical activity (t-test, U-test, ANOVA, Kruskal-Wallis)
  or other computation onto a chart/column in Instant Clue and the underlying dataset
  may contain missing values that the activity cannot handle.
when_to_use_negative:
- When the activity you dispatch explicitly handles missing values — pre-filtering
  is only applied automatically if the activity cannot handle them
- When you intentionally need to impute or otherwise repair missing entries rather
  than drop rows — use the column-based imputation activity instead
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: Desktop GUI that automatically pre-filters rows with missing values before
    submitting data to an activity (such as a statistical test), without altering
    the source data
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:6b6485d20afaca32cd5dee08990770fad7aa41d96881c1d5d4c6e0dff25b88ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# missing-value-prefiltering-before-statistical-activities

## Summary

In Instant Clue, before any statistical test (each test is an activity, e.g. t-test, U-test, ANOVA, Kruskal-Wallis) is executed, rows containing missing values are automatically filtered out for the scope of that activity only, leaving the source data unchanged.

## When to use

Use when dispatching a statistical activity (t-test, U-test, ANOVA, Kruskal-Wallis) or other computation onto a chart/column in Instant Clue and the underlying dataset may contain missing values that the activity cannot handle.

## When NOT to use

- When the activity you dispatch explicitly handles missing values — pre-filtering is only applied automatically if the activity cannot handle them
- When you intentionally need to impute or otherwise repair missing entries rather than drop rows — use the column-based imputation activity instead

## Inputs

- Dataset loaded in Instant Clue (e.g. Tutorial_Data_01.xlsx) with a numerical measurement column and a categorical grouping column (e.g. Condition), possibly containing missing values
- A generated base chart onto which a statistical activity (t-test, U-test, ANOVA, Kruskal-Wallis) is dispatched via drag & drop

## Outputs

- Per-column test statistics and p-values computed on complete-case subsets (annotated on the chart, e.g. p-value above lines connecting tested groups)
- Unmodified source data retained for further analysis and export (e.g. statistical_tests_results.csv)

## How to apply

Load the dataset (e.g. Tutorial_Data_01.xlsx) into Instant Clue and assign the numerical measurement column to the numerical data receiver box and the grouping column (e.g. Condition) to the categorical receiver box by drag & drop. Generate a base chart from these columns, then drag the desired test activity from the Analysis toolbox onto the chart. Because each test is an activity that may not handle missing values, Instant Clue automatically filters the data before submission to the specified activity, without changing the source data — so you should not hand-delete rows before testing. Rationale: per-activity transient filtering guarantees the test runs on complete cases only, while preserving the full dataset for other visualizations, filters, and follow-up activities.

## Related tools

- **Instant Clue** (Desktop GUI that automatically pre-filters rows with missing values before submitting data to an activity (such as a statistical test), without altering the source data) — https://github.com/hnolCol/instantclue

## Evaluation signals

- Every tested column yields a named test with a test statistic and p-value matching its group-comparison condition (two-group vs. multi-group; parametric vs. non-parametric)
- The source dataset still contains its original rows (with missing values) after the test has run — only the activity's internal view was filtered
- Reported p-values equal those obtained from running the same test (e.g. Mann-Whitney U-test or ANOVA) manually on the complete-case subset of the column
- For two-group comparisons, the p-value is annotated on the chart above the lines connecting the tested groups

## Limitations

- Pre-filtering is a removal step, not an imputation step — information from rows with missing values is discarded for that activity; the article notes imputation is available as a separate column activity if repair is preferred
- The paper reports no quantitative acceptance threshold, so correctness is judged qualitatively (each tested column yields a named test with statistic and p-value)

## Evidence

- [results] automatic pre-filtering of missing values: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] statistical tests enabled by drag & drop: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] p-value annotation on chart: [evidence span withheld — anchor status: not_found]
