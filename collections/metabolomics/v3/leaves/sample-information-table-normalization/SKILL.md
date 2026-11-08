---
name: sample-information-table-normalization
description: Use when preparing a sample-information/metadata table (e.g. 3DMouse_duodenum_metadata.txt
  from MSV000083437) before uploading to MassIVE or submitting a GNPS networking job,
  to guarantee schema conformance.
when_to_use_negative:
- Input is already a feature quantification table (e.g. MZmine2 feature table) rather
  than a sample-information file
- The workflow stage is parameter selection or spectral filtering for the networking
  job, not metadata preparation
- Metadata file will not be submitted to GNPS/MassIVE, so the 'filename'/'ATTRIBUTE_'
  schema does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Target platform whose metadata schema ('filename' column, 'ATTRIBUTE_' prefixes)
    the table must satisfy before job submission
- name: MassIVE
  role: Repository where uploaded data files must be name-matched against the 'filename'
    column of the metadata table
- name: pandas
  role: Used to load and validate the tab-separated metadata table during conformance
    checking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:d5a9f4e0174c430daa48c9a93393b5e5813e2ad755627f1808c9e161155f8a08
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
  - MassIVE
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sample-information-table-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates and normalizes the tab-separated sample-information (metadata) table required for GNPS molecular networking jobs, enforcing the 'filename' first-column header, 'ATTRIBUTE_' prefixes on all other columns, and removal of forbidden special characters so metadata can be attached to a submitted dataset.

## When to use

Use when preparing a sample-information/metadata table (e.g. 3DMouse_duodenum_metadata.txt from MSV000083437) before uploading to MassIVE or submitting a GNPS networking job, to guarantee schema conformance.

## When NOT to use

- Input is already a feature quantification table (e.g. MZmine2 feature table) rather than a sample-information file
- The workflow stage is parameter selection or spectral filtering for the networking job, not metadata preparation
- Metadata file will not be submitted to GNPS/MassIVE, so the 'filename'/'ATTRIBUTE_' schema does not apply

## Inputs

- Tab-separated sample-information (metadata) text file, e.g. 3DMouse_duodenum_metadata.txt
- List of uploaded data file names for the MassIVE dataset (e.g. MSV000083437)

## Outputs

- Conformance report (TSV) flagging each row/column schema violation
- Normalized metadata table satisfying the GNPS schema (first column 'filename', other columns 'ATTRIBUTE_'-prefixed, no forbidden characters)

## How to apply

Load the metadata file as a tab-separated table with pandas. Check that the first column header is exactly 'filename' and that every 'filename' cell matches the names of the data files uploaded to MassIVE for the dataset (e.g. MSV000083437). Verify every remaining column header starts with 'ATTRIBUTE_' and that fields contain no forbidden special characters such as '@', '#', '!' or spaces. Flag each row/column violation in a conformance report and write a normalized copy of the table that satisfies the schema; this step sits between uploading data files and submitting the molecular networking job to GNPS.

## Related tools

- **GNPS** (Target platform whose metadata schema ('filename' column, 'ATTRIBUTE_' prefixes) the table must satisfy before job submission)
- **MassIVE** (Repository where uploaded data files must be name-matched against the 'filename' column of the metadata table)
- **pandas** (Used to load and validate the tab-separated metadata table during conformance checking)

## Examples

```
import pandas as pd; df = pd.read_csv('3DMouse_duodenum_metadata.txt', sep='\t'); assert df.columns[0] == 'filename' and all(c.startswith('ATTRIBUTE_') for c in df.columns[1:])
```

## Evaluation signals

- First column header of the normalized table is exactly 'filename'
- Every other column header begins with the prefix 'ATTRIBUTE_'
- Every 'filename' cell matches the name of a data file uploaded to MassIVE for the dataset
- No metadata field contains '@', '#', '!' or spaces
- Conformance report enumerates all original violations and the normalized copy is still tab-separated

## Limitations

- Forbidden special characters and spaces in metadata fields can break GNPS ingestion; spaces act as whitespace-separated token delimiters
- The paper notes standardized sample information is needed to 'reproducibly create publishable molecular networks', but does not give exact per-field length limits
- Metadata normalization alone is qualitative bookkeeping; classic molecular networking provides only qualitative (MS2-based) information

## Evidence
