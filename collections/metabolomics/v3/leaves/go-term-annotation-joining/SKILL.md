---
name: go-term-annotation-joining
description: Use when a loaded data table (Excel, .txt, .csv, .xml, .gz or .zip) already
  carries categorical annotation terms such as GO categories and the goal is to subset
  rows by cellular localization or signaling pathway without altering the source table.
when_to_use_negative:
- The input is already a pre-filtered feature table with no categorical annotation
  columns to join on — no GO/PFAM/MitoCarta term column exists to match against a
  searched category.
- The analysis goal is numerical filtering or statistical testing rather than categorical
  subsetting — use a numerical filter or an activity such as ANOVA/PCA instead of
  'Find Category & Annotate'.
- The task requires modifying the source data in place (e.g. imputation or normalization
  of the full table); categorical annotation joining is deliberately non-destructive
  and does not alter the source table.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: Instant Clue
  role: Provides the 'Find Category & Annotate' categorical filter that joins annotation
    terms to rows and subsets by the searched category, via a drag & drop GUI over
    the loaded data table.
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Scripting language for programmatically reading the .xlsx/.csv table with
    pandas and applying the annotation join plus category match, mirroring the GUI
    filter.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:e0142d80c32e754e75b16a28605cfbf4711c47761b32982dbd565e37be759c7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# go-term-annotation-joining

## Summary

Joins a categorical annotation mapping (e.g. GO terms, PFAM, MitoCarta, GSEA-derived terms) onto a tabular omics dataset and then subsets rows whose annotation term matches a searched category, mirroring Instant Clue's 'Find Category & Annotate' filter.

## When to use

Use when a loaded data table (Excel, .txt, .csv, .xml, .gz or .zip) already carries categorical annotation terms such as GO categories and the goal is to subset rows by cellular localization or signaling pathway without altering the source table.

## When NOT to use

- The input is already a pre-filtered feature table with no categorical annotation columns to join on — no GO/PFAM/MitoCarta term column exists to match against a searched category.
- The analysis goal is numerical filtering or statistical testing rather than categorical subsetting — use a numerical filter or an activity such as ANOVA/PCA instead of 'Find Category & Annotate'.
- The task requires modifying the source data in place (e.g. imputation or normalization of the full table); categorical annotation joining is deliberately non-destructive and does not alter the source table.

## Inputs

- Tabular dataset in Excel (.xlsx), tab-delimited .txt, .csv, .xml, .gz or .zip format with columns carrying categorical annotation terms (e.g. GO terms, PFAM domains, MitoCarta membership, GSEA-derived pathway labels)
- Categorical annotation mapping that links each row (protein/gene) to its category term(s)

## Outputs

- Filtered and annotated table (e.g. filtered_annotated_table.csv) containing only rows whose annotation term matches the searched category, with the annotation/category field intact
- Unmodified source data table (filtering must not change the original data)

## How to apply

Load the data table with pandas/Python (>=3.4) from the 'Tutorial_Data_01.xlsx' Excel file or another supported format (tab-delimited .txt, .csv, .xml, .gz, .zip), reading the columns that carry categorical annotation terms such as GO terms. Join the table with the categorical annotation mapping so that every row carries its category term(s), replicating the annotation step of Instant Clue's first categorical filter, 'Find Category & Annotate'. Apply the find-category filter condition: retain only rows whose category term matches the searched category (e.g. a specific cellular localization or signaling pathway). Importantly, filtering does not modify the source data — Instant Clue automatically pre-filters data before submission to activities without changing the source table, and the annotation label is added to retained rows without altering the original table. Write the filtered and annotated rows to an output CSV so downstream visualization or clustering can operate on the subset.

## Related tools

- **Instant Clue** (Provides the 'Find Category & Annotate' categorical filter that joins annotation terms to rows and subsets by the searched category, via a drag & drop GUI over the loaded data table.) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Scripting language for programmatically reading the .xlsx/.csv table with pandas and applying the annotation join plus category match, mirroring the GUI filter.)

## Examples

```
import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx'); ann = pd.read_csv('go_annotations.csv'); merged = df.merge(ann, on='gene_id'); filtered = merged[merged['GO_category'] == 'mitochondrial respiration']; filtered.to_csv('filtered_annotated_table.csv', index=False)
```

## Evaluation signals

- The output file exists and parses as CSV (file_exists, file_format_is checks).
- The output contains an annotation/category field that matches the annotation source column (field_present check).
- Every retained row's category term matches the searched category — no rows whose annotation term differs from the searched category survive the filter.
- The source data table remains unmodified after filtering (row count and cell values unchanged), respecting the convention that filtering does not alter the source data.
- Expert review confirms the biological correctness of the annotation join — e.g. rows annotated with the correct cellular localization or signaling pathway term.

## Limitations

- Only one of three categorical filters is replicated here ('Find Category & Annotate'); the other two ('Find String(s) & Annotate' and 'Custom Categorical Filter') support string matching and custom rules and are not covered.
- If an activity cannot handle missing values, data are automatically filtered before submission — but this skill does not perform imputation, so rows with missing annotation terms are simply excluded from the subset.
- The panel of statistical tests and further activities for systematic data interpretation are planned extensions in Instant Clue; categorical annotation joining alone does not provide downstream statistical testing.

## Evidence

- [results] There are three different categorical filters: (i) 'Find Category & Annotate': [evidence span withheld — anchor status: not_found]
- [results] The accompanied Tutorial_Data_01.xlsx: [evidence span withheld — anchor status: not_found]
- [results] Instant Clue automatically filters data before submission to activities that cannot handle missing values, without changing the source data: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [readme] README: executables for Windows and Mac are available in the Releases section, and the source code is provided for any platform with Python >=3.4: "Please find the executables for Windows and Mac in the [Releases](https://github.com/hnolCol/instantclue/releases) section."
- [readme] Instant Clue is a Python based desktop application (GUI) using PyQt6 for data visualization and analysis of high dimensional data sets: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
