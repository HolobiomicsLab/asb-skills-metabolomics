---
name: data-annotation-labeling
description: Use when a table must be subset by a categorical annotation term (e.g.
  GO terms, cellular localization, signaling pathway) and matched rows need an added
  category/annotation label while the source table stays unmodified.
when_to_use_negative:
- The data has no categorical annotation terms — use Instant Clue's 'Find String(s)
  & Annotate' or 'Custom Categorical Filter' variants for string or user-defined conditions
  instead.
- The goal is numeric/numerical filtering (e.g. fold-change or p-value cutoffs) rather
  than category-based subsetting — the categorical filter does not apply.
- You intend to overwrite the original source table — the skill's convention, mirroring
  Instant Clue, is that filtering does not modify the source data.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Instant Clue
  role: Source application whose 'Find Category & Annotate' categorical filter this
    skill reproduces; open-source Python desktop GUI (PyQt6) for filtering and annotating
    complex datasets
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Scripted execution environment for loading the xlsx table, joining the annotation
    mapping, applying the category match, and writing the filtered CSV
- name: pandas
  role: Reads the annotation columns of 'Tutorial_Data_01.xlsx' and applies the find-category
    row filter before CSV export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:b426695e4f08a28cecd1acc19fd9794f791a7a1f2480612bbb2ab08378fc3c77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  - pandas
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# data-annotation-labeling

## Summary

Reproduces Instant Clue's 'Find Category & Annotate' categorical filter: subset rows of a table by a categorical annotation term (e.g. GO/PFAM/MitoCarta/GSEA-derived terms) and attach the annotation label to retained rows without modifying the source data.

## When to use

Use when a table must be subset by a categorical annotation term (e.g. GO terms, cellular localization, signaling pathway) and matched rows need an added category/annotation label while the source table stays unmodified.

## When NOT to use

- The data has no categorical annotation terms — use Instant Clue's 'Find String(s) & Annotate' or 'Custom Categorical Filter' variants for string or user-defined conditions instead.
- The goal is numeric/numerical filtering (e.g. fold-change or p-value cutoffs) rather than category-based subsetting — the categorical filter does not apply.
- You intend to overwrite the original source table — the skill's convention, mirroring Instant Clue, is that filtering does not modify the source data.

## Inputs

- Data table with categorical annotation columns, e.g. 'Tutorial_Data_01.xlsx' (Excel/xlsx; also .txt, .csv, .xml, .gz, .zip supported by Instant Clue)
- Categorical annotation mapping (term-to-row mapping, e.g. GO/PFAM/MitoCarta/GSEA-derived term assignments)
- Searched category term (e.g. a specific GO term or cellular localization label)

## Outputs

- filtered_annotated_table.csv — rows matching the searched category with an added annotation/category label field
- Unmodified source table (original rows are not overwritten)

## How to apply

Load the table (e.g. 'Tutorial_Data_01.xlsx' via pandas/python3), reading the columns that carry categorical annotation terms such as GO terms. Join the table with the categorical annotation mapping so every row carries its category term(s), mirroring the annotation step of Instant Clue's 'Find Category & Annotate' filter — the first of its three categorical filters ('Find Category & Annotate', 'Find String(s) & Annotate', 'Custom Categorical Filter'). Apply the find-category condition: retain only rows whose category term matches the searched category (e.g. str.contains or equality on the GO-terms column). Add the annotation label to retained rows without altering the source table, respecting Instant Clue's convention that 'the data are automatically filtered before submission ... without changing the source data'. Write the filtered and annotated rows to filtered_annotated_table.csv and validate that the file exists, parses as CSV, contains an annotation/category field, and that every retained row matches the searched category.

## Related tools

- **Instant Clue** (Source application whose 'Find Category & Annotate' categorical filter this skill reproduces; open-source Python desktop GUI (PyQt6) for filtering and annotating complex datasets) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Scripted execution environment for loading the xlsx table, joining the annotation mapping, applying the category match, and writing the filtered CSV)
- **pandas** (Reads the annotation columns of 'Tutorial_Data_01.xlsx' and applies the find-category row filter before CSV export)

## Examples

```
import pandas as pd; df = pd.read_excel('Tutorial_Data_01.xlsx'); hit = df[df['GO_terms'].str.contains('mitochondrion', na=False)].assign(annotation='mitochondrion'); hit.to_csv('filtered_annotated_table.csv', index=False)
```

## Evaluation signals

- filtered_annotated_table.csv exists on disk and parses as a valid CSV file (file_exists, file_format_is).
- Output contains an annotation/category field (field_present).
- Every retained row's category term matches the searched category — no non-matching rows leak through (expert_review on biological correctness of the annotation join).
- Source table is unchanged after the operation (row count and content identical to input).
- Rows absent from the output are exactly those whose category terms do not match the searched term.

## Limitations

- Correctness depends entirely on the annotation-mapping join: rows lacking category terms (e.g. missing GO annotations) will be dropped and must be handled explicitly.
- Instant Clue provides three distinct categorical filters ('Find Category & Annotate', 'Find String(s) & Annotate', 'Custom Categorical Filter'); this skill only mirrors the category variant — string search or custom conditions need different logic.
- When installing Instant Clue from source, the combat package has 'very specific versioning requirements of pandas and numpy' and is not in requirements.txt, so the Python environment for scripted annotation may need combat installed first with a pandas/numpy version-mismatch warning ignored.

## Evidence

- [results] There are three different categorical filters: (i) 'Find Category & Annotate': [evidence span withheld — anchor status: not_found]
- [results] the data are automatically filtered before submission to the specified activity, without changing the source data: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv): "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv)"
- [readme] Then create a virtual environment to not mix up required package versions with your python installation: "Then create a virtual environment to not mix up required package versions with your python installation"
