---
name: technical-replicate-metadata-mapping
description: 'Use when starting an mpactR workflow: you have a Progenesis peak table
  CSV and a sample metadata CSV with Injection, Sample_Code, and Biological_Group
  columns, and technical replicates must be linked to biological groups before filtering.'
when_to_use_negative:
- Your peak table is not from Progenesis pre-processing (e.g. MS-DIAL or Bruker Metaboscape
  exports handled by the Python MPACT tool) — import_data with format = "Progenesis"
  expects Progenesis-format columns.
- You only need to run filters without replicate-aware grouping — filters such as
  filter_mispicked_ions() are independent of metadata grouping.
- Metadata lacks the required Injection, Sample_Code, and Biological_Group columns
  — the downstream CV filter between technical replicates cannot be applied correctly.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: Provides import_data() to build the R6 object from peak table and metadata,
    and get_metadata()/get_raw_data()/get_peak_table() accessors for validation
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime language for mpactr; the imported object is a data.table-backed R6
    instance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:0f0554b078a929c25d10fdfe7e851e61065fa5725414d2e19069b8d34c91ba95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# technical-replicate-metadata-mapping

## Summary

Ingest a Progenesis-format LC-MS/MS peak table and its sample metadata CSV into an mpactR R6 object so that each injection (including technical replicates) is mapped to its Sample_Code and Biological_Group, enabling downstream replicate-reproducibility (CV) filtering.

## When to use

Use when starting an mpactR workflow: you have a Progenesis peak table CSV and a sample metadata CSV with Injection, Sample_Code, and Biological_Group columns, and technical replicates must be linked to biological groups before filtering.

## When NOT to use

- Your peak table is not from Progenesis pre-processing (e.g. MS-DIAL or Bruker Metaboscape exports handled by the Python MPACT tool) — import_data with format = "Progenesis" expects Progenesis-format columns.
- You only need to run filters without replicate-aware grouping — filters such as filter_mispicked_ions() are independent of metadata grouping.
- Metadata lacks the required Injection, Sample_Code, and Biological_Group columns — the downstream CV filter between technical replicates cannot be applied correctly.

## Inputs

- Progenesis-format LC-MS/MS peak table CSV (e.g. cultures_peak_table.csv with compound, m/z, retention time, and raw abundance columns)
- Sample metadata CSV (e.g. cultures_metadata.csv with required Injection, Sample_Code, and Biological_Group columns)

## Outputs

- mpactR R6 class object storing both the peak table and metadata (peak table retrievable via get_peak_table())
- Metadata as a data.table (via get_metadata()) with Injection, Sample_Code, Biological_Group columns
- Unfiltered Progenesis peak table as a data.table (via get_raw_data())
- Serialised RDS file (e.g. imported_data.rds)

## How to apply

Load the mpactr package and resolve the bundled example files via example_path() for cultures_peak_table.csv and cultures_metadata.csv, then call import_data() with peak_table_file_path and metadata_file_path and format = "Progenesis". This creates an R6 class object that stores both the peak table and the metadata, providing the structure through which technical replicates are tied to their sample codes and biological groups. Validate the import by retrieving the metadata with get_metadata() — confirming the required Injection, Sample_Code, and Biological_Group columns are present so technical replicates can be distinguished — and the unfiltered Progenesis peak table (compound, m/z, retention time, and raw abundance columns) with get_raw_data(); both accessors return data.table objects. Correct replicate-to-group mapping at this stage is critical because filter_cv() identifies non-reproducible ions by coefficient of variation (CV) computed between technical replicates. Persist the validated object (e.g. as imported_data.rds) before running any of the independent filters.

## Related tools

- **mpactr** (Provides import_data() to build the R6 object from peak table and metadata, and get_metadata()/get_raw_data()/get_peak_table() accessors for validation) — https://github.com/mums2/mpactr
- **R** (Runtime language for mpactr; the imported object is a data.table-backed R6 instance)

## Examples

```
data2 <- mpactr::import_data(peak_table_file_path = example_path("cultures_peak_table.csv"), metadata_file_path = example_path("cultures_metadata.csv"), format = "Progenesis"); get_metadata(data2)
```

## Evaluation signals

- get_metadata() returns a data.table containing the required Injection, Sample_Code, and Biological_Group columns.
- get_raw_data() returns the unfiltered Progenesis peak table as a data.table with compound, m/z, retention time, and raw abundance columns.
- get_peak_table(data2) retrieves the peak table from the R6 object without error.
- Every sample row in the metadata can be matched to the peak table columns, so technical replicates map to a Biological_Group for filter_cv().
- The R6 object round-trips correctly when saved to imported_data.rds and reloaded.

## Limitations

- import_data with format = "Progenesis" assumes Progenesis pre-processing; other peak-list formats are not covered by this skill.
- Metadata must explicitly carry Injection, Sample_Code, and Biological_Group columns; missing columns will silently break the replicate-to-group mapping that filter_cv() depends on.
- All mpactr filters are independent, so mis-mapped replicates will not be caught by other filters and only surface in CV-based reproducibility results.

## Evidence

- [methods] To import these data into R, use the mpactr function `import_data()`: "To import these data into R, use the mpactr function `import_data()`"
- [methods] Non-reproducible ions are identified by coefficient of variation (CV) between technical replicates with `filter_cv()`: "Non-reproducible ions are identified by coefficient of variation (CV) between technical replicates with `filter_cv()`"
- [methods] Metadata can be accessed with `get_metadata()`: "Metadata can be accessed with `get_metadata()`"
- [readme] `filter_cv()`: removal of non-reproducible features, or those that are inconsistent between technical replicates.: [evidence span withheld — anchor status: external_doc]
