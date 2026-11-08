---
name: tsv-file-parsing
description: Use when a metadata/sample-information table must be parsed or validated
  before submitting a GNPS molecular networking job, or when diagnosing why a deposited
  dataset's metadata fails the GNPS schema.
when_to_use_negative:
- Input is already a processed feature quantification table (e.g. MZmine2 output)
  rather than a sample-information/metadata table
- The goal is parsing the molecular networking results table rather than preparing
  metadata for a GNPS job submission
- The file is genuinely comma-separated CSV with a different delimiter — splitting
  on tabs would corrupt fields; convert delimiter handling first
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Defines the metadata schema (tab-separated table, 'filename' first column,
    'ATTRIBUTE_'-prefixed columns) that the parsed table must conform to; the metadata
    is attached to molecular networking job submissions
  repo: http://gnps.ucsd.edu
- name: pandas
  role: Used to load the deposited metadata file as a tab-separated table and inspect
    headers/fields for schema violations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:4a19277d94dcbfe645a25e3518466d57bc9e512b11884f1838ef83f60d697bdf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tsv-file-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load and validate tab-separated sample-information (metadata) tables required for GNPS molecular networking jobs, checking the GNPS schema: a 'filename' first column and 'ATTRIBUTE_'-prefixed remaining columns.

## When to use

Use when a metadata/sample-information table must be parsed or validated before submitting a GNPS molecular networking job, or when diagnosing why a deposited dataset's metadata fails the GNPS schema.

## When NOT to use

- Input is already a processed feature quantification table (e.g. MZmine2 output) rather than a sample-information/metadata table
- The goal is parsing the molecular networking results table rather than preparing metadata for a GNPS job submission
- The file is genuinely comma-separated CSV with a different delimiter — splitting on tabs would corrupt fields; convert delimiter handling first

## Inputs

- Sample-information (metadata) file as tab-separated text, e.g. 3DMouse_duodenum_metadata.txt from MSV000083437
- List of data file names uploaded to the MassIVE dataset

## Outputs

- Schema conformance report (TSV) listing flagged row/column violations
- Normalized metadata table (TSV) that satisfies the GNPS schema

## How to apply

Read the sample-information file as a tab-separated text table (e.g. with pandas read_csv(..., sep='\t')), never as comma-separated. Verify the first column header is exactly 'filename' and that every value in that column matches the names of the data files uploaded to MassIVE for the dataset (e.g. MSV000083437). Check that every remaining column header begins with 'ATTRIBUTE_' and that no metadata field contains forbidden characters such as '@', '#', '!' or spaces. Flag each violating row or column in a conformance report and write a normalized TSV copy that satisfies the schema.

## Related tools

- **GNPS** (Defines the metadata schema (tab-separated table, 'filename' first column, 'ATTRIBUTE_'-prefixed columns) that the parsed table must conform to; the metadata is attached to molecular networking job submissions) — http://gnps.ucsd.edu
- **pandas** (Used to load the deposited metadata file as a tab-separated table and inspect headers/fields for schema violations)

## Examples

```
import pandas as pd; df = pd.read_csv('3DMouse_duodenum_metadata.txt', sep='\t'); assert df.columns[0] == 'filename' and all(c.startswith('ATTRIBUTE_') for c in df.columns[1:])
```

## Evaluation signals

- First column header of the parsed table is exactly 'filename'
- Every 'filename' cell value matches a data file name uploaded to MassIVE for the dataset
- All remaining column headers start with the 'ATTRIBUTE_' prefix
- No metadata field contains '@', '#', '!', or space characters
- The normalized output round-trips as a tab-separated text file with unchanged column count

## Limitations

- Whitespace-separated tokens in field values count as violations; fields must avoid spaces, which can require renaming entries before submission
- Correct delimiter handling is essential: the metadata must be tab-separated text, so parsing it as comma-separated would silently misparse the table

## Evidence

- [supplementary] The metadata table must be a tab-separated text file whose first column header is 'filename' matching uploaded data file names, with every other column header starting with 'ATTRIBUTE_': [evidence span withheld — anchor status: not_found]
- [readme] GNPS is a web-based mass spectrometry ecosystem for sharing and organizing tandem MS data: [evidence span withheld — anchor status: external_doc]
