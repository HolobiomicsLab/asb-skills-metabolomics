---
name: metadata-conformant-peak-table-import
description: Use when raw tandem MS/MS data have already been preprocessed by a peak-picking
  tool (e.g. Progenesis) and you need to load the resulting peak table plus a sample-metadata
  file into R as an mpactr object before running filters.
when_to_use_negative:
- Raw unprocessed vendor files (e.g. Bruker Metaboscape or MS-DIAL exports for the
  Python MPACT tool) rather than a supported preprocessed peak-table format for import_data()
- You only need the filtered peak table after filtering has already been run — use
  get_peak_table() instead of re-importing
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mpactr
  role: R package whose import_data() function loads the Progenesis peak table and
    metadata into an mpactr object
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment for the mpactr package and import_data() call
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:8ad706071fb14ee19e2d55d62f443d17205cbef2ebbee7215a0e323cb35eceda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metadata-conformant-peak-table-import

## Summary

Import a preprocessed LC-MS/MS peak table (e.g. a Progenesis export such as the PTY087I2 dataset) together with its sample metadata into mpactr's R environment using import_data(), creating the mpactr object on which all downstream filters operate.

## When to use

Use when raw tandem MS/MS data have already been preprocessed by a peak-picking tool (e.g. Progenesis) and you need to load the resulting peak table plus a sample-metadata file into R as an mpactr object before running filters.

## When NOT to use

- Raw unprocessed vendor files (e.g. Bruker Metaboscape or MS-DIAL exports for the Python MPACT tool) rather than a supported preprocessed peak-table format for import_data()
- You only need the filtered peak table after filtering has already been run — use get_peak_table() instead of re-importing

## Inputs

- Progenesis-format peak table file (peak_table_file_path) exported from tandem MS/MS preprocessing
- Sample metadata file (metadata_file_path) mapping samples to biological groups and technical replicates
- Format specification string (e.g. "Progenesis")

## Outputs

- mpactr object holding the imported peak table and metadata
- Metadata data frame accessible via get_metadata()

## How to apply

Call mpactr's import_data() with the path to the peak table file, the path to the metadata file, and format = "Progenesis" so the parser matches the Progenesis column layout of the exported peak table. The metadata file must map each sample (including biological groups and technical replicates, as in the 38-sample PTY087I2 dataset with solvent blanks, media blanks, and 0um_Ce/250um_Ce Streptomyces sp. PTY087I2 conditions) to its column in the peak table, because filters such as filter_group() and filter_cv() rely on group labels. import_data() returns an mpactr object that carries both the peak table and the metadata, which can be verified with get_metadata() before any filtering. Choosing the correct format string is the key decision point: an unsupported or mismatched export format will fail to align ions with samples.

## Related tools

- **mpactr** (R package whose import_data() function loads the Progenesis peak table and metadata into an mpactr object) — https://github.com/mums2/mpactr
- **R** (Runtime environment for the mpactr package and import_data() call)

## Examples

```
library(mpactr); mpactr_object <- import_data(peak_table_file_path, metadata_file_path, format = "Progenesis")
```

## Evaluation signals

- import_data() returns an mpactr object without format-mismatch errors
- get_metadata() reproduces the expected sample count and group labels (e.g. 38 samples across solvent blanks, media blanks, 0um_Ce, 250um_Ce for PTY087I2)
- Subsequent filter steps (filter_mispicked_ions, filter_group, filter_cv) can access sample groups from the imported object, e.g. filter_group(group_threshold = 0.01, group_to_remove = "Blanks") finds the Blanks group
- Ion counts in the imported peak table match the pre-filter counts later reported by filter_summary() (e.g. 2774 ions prior to mispicked filtering in the PTY087I2 example)

## Limitations

- Only the preprocessing formats supported by import_data()'s format argument (e.g. Progenesis) can be imported; other vendor exports are not handled by mpactr
- Metadata must be supplied separately; mpactr does not infer biological groups or technical-replicate structure from the peak table alone
- The article's example workflow applies import_data() before any filtering; filters are independent but each expects the mpactr object produced by import_data()

## Evidence

- [methods] To import these data into R, use the mpactr function `import_data()`: "To import these data into R, use the mpactr function `import_data()`"
- [methods] mpactr corrects errors from preprocessing of raw tandem MS/MS data: "The goal of mpactr is to correct for errors that occur during the pre-processing of raw tandem MS/MS data"
- [methods] This dataset contain 38 samples for biological groups solvent blanks, media blanks, Streptomyces sp. PTY08712 grown with (250um_Ce) and without (0um_Ce) rare earth element cerium: "This dataset contain 38 samples for biological groups solvent blanks, media blanks, Streptomyces sp. PTY08712 grown with (250um_Ce)"
- [methods] Metadata can be accessed with `get_metadata()`: "Metadata can be accessed with `get_metadata()`"
- [readme] removal of features overrepresented in a specific group of samples; for example removal of features present in solvent blanks due to carryover between samples: [evidence span withheld — anchor status: external_doc]
