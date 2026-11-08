---
name: gnps-metadata-schema-validation
description: Use when preparing to submit a dataset (e.g. MSV000083437) to MassIVE
  or launch a GNPS molecular networking job and the metadata table must be checked/normalized
  to the GNPS sample-information schema.
when_to_use_negative:
- Input is already a normalized feature quantification table or a GNPS job output
  rather than a raw sample-information table.
- Metadata is destined for a different repository (e.g. Metabolomics Workbench or
  MetaboLights) whose sample-information schema differs from GNPS/MassIVE.
- The goal is spectral processing or peak filtering (e.g. precursor ion window or
  50 Da window filters) rather than metadata validation.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Target platform whose sample-information (metadata) schema the table must
    satisfy; metadata is uploaded alongside the dataset to MassIVE and used when submitting
    a molecular networking job
  repo: http://gnps.ucsd.edu
- name: pandas
  role: Loads the tab-separated metadata file for header- and value-level schema checking
    and normalization
- name: CCMS-UCSD/GNPSDocumentation
  role: Authoritative documentation of the GNPS metadata schema and workflow requirements
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:05fc77ac804e8ac860989a6a778e78968d80e6fbbb0bad3cf31bdf7cf08653e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - CCMS-UCSD/GNPSDocumentation
  - GNPS
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gnps-metadata-schema-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates and normalizes a sample-information (metadata) table so it conforms to the GNPS/MassIVE schema required before submitting a molecular networking job, ensuring metadata can be joined to uploaded LC-MS/MS data files.

## When to use

Use when preparing to submit a dataset (e.g. MSV000083437) to MassIVE or launch a GNPS molecular networking job and the metadata table must be checked/normalized to the GNPS sample-information schema.

## When NOT to use

- Input is already a normalized feature quantification table or a GNPS job output rather than a raw sample-information table.
- Metadata is destined for a different repository (e.g. Metabolomics Workbench or MetaboLights) whose sample-information schema differs from GNPS/MassIVE.
- The goal is spectral processing or peak filtering (e.g. precursor ion window or 50 Da window filters) rather than metadata validation.

## Inputs

- Tab-separated sample-information (metadata) text file (e.g. 3DMouse_duodenum_metadata.txt from MSV000083437)
- List of data file names uploaded to MassIVE for the dataset

## Outputs

- Conformance report (TSV) listing each metadata row/column schema violation
- Normalized metadata table (tab-separated) with 'filename' first column and 'ATTRIBUTE_'-prefixed headers

## How to apply

Load the metadata file (e.g. 3DMouse_duodenum_metadata.txt deposited for MSV000083437) with pandas as a tab-separated text table. Check that the first column header is exactly 'filename' and that every 'filename' cell matches the names of the data files uploaded to MassIVE. Verify that all remaining column headers are prefixed with 'ATTRIBUTE_' and contain no forbidden special characters or spaces (e.g. '@', '#', '!', whitespace-separated tokens). Flag each violating row or column in a conformance report, then write a normalized TSV copy of the table that satisfies the schema. This step corresponds to the article's workflow stage of collating and uploading a standardized sample-information (metadata) table before submitting the GNPS job.

## Related tools

- **GNPS** (Target platform whose sample-information (metadata) schema the table must satisfy; metadata is uploaded alongside the dataset to MassIVE and used when submitting a molecular networking job) — http://gnps.ucsd.edu
- **pandas** (Loads the tab-separated metadata file for header- and value-level schema checking and normalization)
- **CCMS-UCSD/GNPSDocumentation** (Authoritative documentation of the GNPS metadata schema and workflow requirements) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
import pandas as pd; df = pd.read_csv('3DMouse_duodenum_metadata.txt', sep='\t'); assert df.columns[0] == 'filename' and all(c.startswith('ATTRIBUTE_') for c in df.columns[1:])
```

## Evaluation signals

- First column header of the normalized table is exactly 'filename'.
- Every 'filename' cell matches a data file name uploaded to MassIVE for the dataset (no orphan rows or missing files).
- All non-first column headers begin with the 'ATTRIBUTE_' prefix.
- No metadata field contains forbidden special characters ('@', '#', '!') or spaces / whitespace-separated tokens.
- Conformance report (TSV) accounts for every flagged row/column violation found in the original file.

## Limitations

- The validation checks header names, filename matching, and forbidden characters; it does not validate biological content or statistics of the metadata (the tutorial example had no statistics performed and was not analyzed blindly).
- Applies specifically to the GNPS/MassIVE standardized metadata format; other repositories use different schemas.

## Evidence

- [intro] Collate and upload a sample information (metadata) table: "reproducibly create publishable molecular networks using standardized sample information (metadata)"
- [intro] Workflow step ordering for metadata upload: "(iii) uploading data to the MassIVE public repository"
- [readme] Metadata schema documented in GNPS docs: "**GNPS_Documentation** GitHub respository that points to the documentation website [(https://ccms-ucsd.github.io/GNPSDocumentation)]"
- [supplementary] Dataset example used for validation: "MSV000083437 (germ-free (GF) and speci fi c-pathogen-free (SPF) mice)"
