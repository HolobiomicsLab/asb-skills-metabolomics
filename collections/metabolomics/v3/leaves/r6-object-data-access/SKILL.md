---
name: r6-object-data-access
description: Use when an mpactR R6 object exists (after import_data) and you need
  to retrieve the raw or filtered Progenesis-format peak table, verify sample metadata
  columns, extract CV values from filter_cv, or compute group means.
when_to_use_negative:
- The input is still a raw Progenesis CSV or MS-DIAL/GNPS export that has not yet
  been passed through mpactR::import_data() — no R6 object exists to access.
- The goal is to apply or change filtering behavior (mispicked ions, blanks, CV, insource
  ions) — accessors only retrieve results, they do not run filters.
- Working in the original Python MPACT (BalunasLab/mpact) workflow rather than the
  R package, which does not expose R6 accessor methods.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: Provides the R6 object and its accessor methods (get_peak_table, get_metadata,
    get_raw_data, get_cv_data, get_group_averages) used for data retrieval
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment in which the mpactR R6 object and data.table outputs are
    created and consumed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:3a5e8bc418712c2374d67bc3669838fff6564c4a300611097083596db5bf6d62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# r6-object-data-access

## Summary

Extracts peak tables, sample metadata, CV values, and group means from the mpactR R6 object using its accessor methods (get_peak_table, get_metadata, get_raw_data, get_cv_data, get_group_averages), which return data.table objects ready for downstream inspection or export.

## When to use

Use when an mpactR R6 object exists (after import_data) and you need to retrieve the raw or filtered Progenesis-format peak table, verify sample metadata columns, extract CV values from filter_cv, or compute group means.

## When NOT to use

- The input is still a raw Progenesis CSV or MS-DIAL/GNPS export that has not yet been passed through mpactR::import_data() — no R6 object exists to access.
- The goal is to apply or change filtering behavior (mispicked ions, blanks, CV, insource ions) — accessors only retrieve results, they do not run filters.
- Working in the original Python MPACT (BalunasLab/mpact) workflow rather than the R package, which does not expose R6 accessor methods.

## Inputs

- mpactR R6 class object created by import_data() from a Progenesis-format peak table CSV and sample metadata CSV
- accessor method name to invoke (get_peak_table, get_metadata, get_raw_data, get_cv_data, get_group_averages)

## Outputs

- filtered peak table (data.table) with the applied filters reflected
- sample metadata (data.table) with Injection, Sample_Code, Biological_Group columns
- raw unfiltered Progenesis peak table (data.table) with compound, m/z, retention time, and raw abundance columns
- CV values (data.table) from filter_cv()
- group means (data.table) from get_group_averages()

## How to apply

After calling mpactr::import_data() on the Progenesis peak table and metadata CSVs (format = "Progenesis"), interact with the resulting R6 object only through its accessor methods rather than touching internals. Call get_metadata() to confirm the required Injection, Sample_Code, and Biological_Group columns are present, and get_raw_data() to confirm the unfiltered table retains compound, m/z, retention time, and raw abundance columns. After filters run, use get_peak_table() to extract the filtered peak table with filters applied, get_cv_data() to recover the CV values calculated during filter_cv(), and get_group_averages() to extract group means from filtered compounds. All accessors return data.table objects, so validate structure and dimensions with standard data.table checks before saving (e.g. as imported_data.rds).

## Related tools

- **mpactr** (Provides the R6 object and its accessor methods (get_peak_table, get_metadata, get_raw_data, get_cv_data, get_group_averages) used for data retrieval) — https://github.com/mums2/mpactr
- **R** (Runtime environment in which the mpactR R6 object and data.table outputs are created and consumed)

## Examples

```
data2 <- mpactr::import_data(peak_table_file_path, metadata_file_path, format = "Progenesis"); get_peak_table(data2)
```

## Evaluation signals

- get_metadata() returns a data.table containing the required Injection, Sample_Code, and Biological_Group columns.
- get_raw_data() returns the unfiltered Progenesis table with compound, m/z, retention time, and raw abundance columns.
- get_peak_table() called after filtering returns a table whose ion set excludes ions that failed the applied filters (e.g. mispicked, blank, or CV-failing ions).
- get_cv_data() returns CV values consistent with those calculated during filter_cv().
- All accessor outputs are valid data.table objects and can be serialized (e.g. saved as imported_data.rds).

## Limitations

- Accessors only retrieve data stored in the R6 object; the filtered peak table content depends on which of the independent filters were run beforehand.
- Outputs are data.table objects, so downstream code must be R/data.table compatible.
- The article's discussion notes no changelog was found, so accessor behavior across package versions is not documented here.

## Evidence

- [methods] Metadata can be accessed with `get_metadata()`: "Metadata can be accessed with `get_metadata()`"
- [methods] extract the filtered peak table: "If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`"
- [methods] extract group means: "We can extract group means from the filtered compounds using the mpactr function `get_group_averages()`"
- [methods] extract the CV calculated during filter_cv: "you can extract the CV calculated during `filer_cv()` using mpactr's `get_cv_data()` function"
- [readme] mpactr purpose: "mpactr is a collection of filters for the purpose of identifying high
quality MS1 features by correcting peak selection errors introduced
during the pre-processing of tandem mass spectrometry data."
