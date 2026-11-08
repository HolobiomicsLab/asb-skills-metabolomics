---
name: table-row-subsetting
description: Use when a table carries per-row categorical annotation terms (e.g. cellular
  localization, signaling pathway) and the goal is to retain only rows matching a
  searched category, then attach the annotation label to the subset.
when_to_use_negative:
- The subset should be defined by graphical selection in a scatter plot rather than
  by a category term — Instant Clue's selection tool is the appropriate route for
  interactive subsetting
- The filtering criterion is numeric (e.g. fold-change or p-value thresholds) rather
  than categorical; the numerical filtering path applies instead
- The source table must itself be modified in place — the convention here is that
  filtering does not alter the source data
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Source application whose 'Find Category & Annotate' categorical filter this
    subsetting procedure reproduces; categorization/filter/annotation functions on
    complex datasets
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for the pandas-based loading, annotation join, category filter and
    CSV export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:b52ca342cbf32901d4aad11206e30404bb32e2d676bd80c6e4886f3c57e2a5b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# table-row-subsetting

## Summary

Subsets rows of an annotation-bearing data table by categorical criteria (e.g. GO, PFAM, MitoCarta or GSEA-derived terms), mirroring Instant Clue's 'Find Category & Annotate' categorical filter, and writes the retained rows to a new file without altering the source table.

## When to use

Use when a table carries per-row categorical annotation terms (e.g. cellular localization, signaling pathway) and the goal is to retain only rows matching a searched category, then attach the annotation label to the subset.

## When NOT to use

- The subset should be defined by graphical selection in a scatter plot rather than by a category term — Instant Clue's selection tool is the appropriate route for interactive subsetting
- The filtering criterion is numeric (e.g. fold-change or p-value thresholds) rather than categorical; the numerical filtering path applies instead
- The source table must itself be modified in place — the convention here is that filtering does not alter the source data

## Inputs

- Excel data table (e.g. Tutorial_Data_01.xlsx) with columns carrying categorical annotation terms (GO terms, PFAM, MitoCarta, GSEA-derived terms)
- Categorical annotation mapping joining rows to their category term(s)

## Outputs

- filtered_annotated_table.csv containing only rows matching the searched category, with the annotation/category field attached

## How to apply

Load the table from the Excel file (e.g. 'Tutorial_Data_01.xlsx') with pandas/Python >=3.4, reading the columns that carry categorical annotation terms such as GO terms. Join the table with the categorical annotation mapping so every row carries its category term(s), mirroring the annotation step of Instant Clue's first of three categorical filters ('Find Category & Annotate'). Apply the find-category condition: retain only rows whose category term matches the searched category. Add the annotation label to the retained rows while leaving the source table unmodified, respecting Instant Clue's convention that filtering does not change the source data. Write the filtered and annotated rows to filtered_annotated_table.csv and verify it parses as CSV with an annotation/category field present.

## Related tools

- **Instant Clue** (Source application whose 'Find Category & Annotate' categorical filter this subsetting procedure reproduces; categorization/filter/annotation functions on complex datasets) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for the pandas-based loading, annotation join, category filter and CSV export)

## Examples

```
python3 -c "import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx'); df[df['GO_terms'].str.contains('nucleus', na=False)].assign(annotation='nucleus').to_csv('filtered_annotated_table.csv', index=False)"
```

## Evaluation signals

- filtered_annotated_table.csv exists on disk
- The output parses as a valid CSV file
- The output contains an annotation/category field
- Every retained row's category term matches the searched category (no off-category rows leak through)
- Expert review confirms the biological correctness of the annotation join

## Limitations

- Instant Clue implements three categorical filters ('Find Category & Annotate', 'Find String(s) & Annotate', 'Custom Categorical Filter'); this skill covers only the first — string-match and custom-rule subsetting follow different filters
- Filtering must not modify the source data; activities that cannot handle missing values trigger automatic pre-filtering before submission instead
- When running Instant Clue from source, the combat package is excluded from requirements.txt due to specific pandas/numpy versioning constraints and must be installed first, ignoring the numpy/pandas version-mismatch warning

## Evidence

- [results] find-category filter condition: [evidence span withheld — anchor status: not_found]
- [results] no source-data mutation: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] filter entry point: "The filter button can be used to quickly apply categorical and numerical filtering"
- [results] GUI alternative for subsetting: "The selection tool allows the user to select data in a scatter plot for sub setting and further analysis"
- [readme] running from source: "Then use the requirements.txt file to install required packages and finally start InstantClue."
