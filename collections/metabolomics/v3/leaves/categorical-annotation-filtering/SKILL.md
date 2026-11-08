---
name: categorical-annotation-filtering
description: Use when a table carries categorical annotation columns (e.g. GO terms,
  cellular localization, signaling pathway) and rows must be retained only if their
  category term matches a searched category, then annotated.
when_to_use_negative:
- The goal is substring search over free-text rather than category membership — use
  Instant Clue's 'Find String(s) & Annotate' filter instead
- Filtering criteria are user-defined numerical or custom rules rather than a categorical
  annotation term — 'Custom Categorical Filter' or numerical filtering is the appropriate
  mechanism
- The input lacks any categorical annotation columns to match against (no GO/PFAM/pathway/localization
  terms)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Source application providing the 'Find Category & Annotate' categorical filter
    that this skill reproduces programmatically
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for loading the Excel table with pandas and executing the filter/annotate/export
    steps
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:1ad8e0ce1f389bf13718a9b8fa785f429b7cfbdd0d5cb6f6af4bb4492c93f297
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# categorical-annotation-filtering

## Summary

Implements Instant Clue's 'Find Category & Annotate' categorical filter: subset rows of an annotated table (e.g. GO, PFAM, MitoCarta or GSEA-derived term columns) by matching a searched category term and add an annotation label without altering the source data.

## When to use

Use when a table carries categorical annotation columns (e.g. GO terms, cellular localization, signaling pathway) and rows must be retained only if their category term matches a searched category, then annotated.

## When NOT to use

- The goal is substring search over free-text rather than category membership — use Instant Clue's 'Find String(s) & Annotate' filter instead
- Filtering criteria are user-defined numerical or custom rules rather than a categorical annotation term — 'Custom Categorical Filter' or numerical filtering is the appropriate mechanism
- The input lacks any categorical annotation columns to match against (no GO/PFAM/pathway/localization terms)

## Inputs

- Data table with categorical annotation columns (e.g. Excel file 'Tutorial_Data_01.xlsx' with GO terms)
- Categorical annotation mapping (term-to-row join, e.g. GO/PFAM/MitoCarta/GSEA-derived terms)
- Searched category term (e.g. a cellular localization or signaling pathway label)

## Outputs

- filtered_annotated_table.csv containing only rows whose category term matches the searched category, with an annotation/category field

## How to apply

Load the data table (e.g. 'Tutorial_Data_01.xlsx') with pandas/Python >=3.4, reading the columns that carry categorical annotation terms such as GO terms. Join the table with the categorical annotation mapping so every row carries its category term(s), mirroring the annotation step of Instant Clue's 'Find Category & Annotate' filter — the first of its three categorical filters alongside 'Find String(s) & Annotate' and 'Custom Categorical Filter'. Apply the find-category condition: retain only rows whose category term matches the searched category (e.g. a cellular localization or pathway term). Add the annotation label to the retained rows while leaving the source table unmodified, respecting Instant Clue's convention that filtering does not change the source data. Write the filtered and annotated rows to filtered_annotated_table.csv and validate that the file exists, parses as CSV, contains an annotation/category field, and that every retained row matches the searched category term.

## Related tools

- **Instant Clue** (Source application providing the 'Find Category & Annotate' categorical filter that this skill reproduces programmatically) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for loading the Excel table with pandas and executing the filter/annotate/export steps) — https://github.com/hnolCol/instantclue

## Examples

```
import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx'); hit = df[df['GO_terms'].str.contains(searched_category, na=False)]; hit.assign(annotation=searched_category).to_csv('filtered_annotated_table.csv', index=False)
```

## Evaluation signals

- filtered_annotated_table.csv exists on disk
- The output parses as valid CSV
- The output contains an annotation/category field
- Every retained row's category term matches the searched category (biological correctness of the annotation join confirmed by expert review)
- The source data table remains unmodified after filtering

## Limitations

- Only one of three categorical filters in Instant Clue; string-search and custom categorical criteria require the sibling filters
- If a downstream activity cannot handle missing values, data are automatically pre-filtered, which can silently change the row set being annotated
- Exact category matching depends on the annotation mapping join being correct; a mis-join will produce rows annotated with the wrong term

## Evidence

- [results] Find Category & Annotate filter definition: [evidence span withheld — anchor status: not_found]
- [results] Filtering does not modify source data: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Tutorial dataset input: [evidence span withheld — anchor status: not_found]
- [intro] Objective to filter, annotate and select data: "(iv) Availability of functions to filter, annotate and select data from complex datasets."
- [intro] Python runtime requirement: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
