---
name: massive-dataset-metadata-matching
description: Use when preparing to submit a GNPS molecular networking job (e.g., METABOLOMICS-SNETS-V2
  or FBMN) for a dataset such as MSV000083437 and a sample-information table must
  be collated and matched to the uploaded data files.
when_to_use_negative:
- Input is already a feature table or quantification table (e.g., MZmine2 output)
  rather than a sample-information metadata sheet
- The dataset files have not yet been uploaded to MassIVE, so filename matching against
  repository contents cannot be performed
- The goal is a follow-up targeted LC-MS quantitation workflow rather than metadata
  preparation for molecular networking submission
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Platform whose metadata schema ('filename' first column, 'ATTRIBUTE_' column
    prefix) the table must satisfy before molecular networking job submission
  repo: http://gnps.ucsd.edu
- name: MassIVE
  role: Repository holding the uploaded LC-MS data files whose names must be matched
    by the metadata 'filename' column
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:7124bd9a5be0d2a3fa6225a9badd80591ace97d888486bc537c9a60841330211
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# MassIVE dataset metadata matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and validates the sample-information (metadata) table required for GNPS molecular networking jobs, ensuring the 'filename' column matches uploaded MassIVE data files and remaining columns carry the 'ATTRIBUTE_' prefix with no forbidden characters.

## When to use

Use when preparing to submit a GNPS molecular networking job (e.g., METABOLOMICS-SNETS-V2 or FBMN) for a dataset such as MSV000083437 and a sample-information table must be collated and matched to the uploaded data files.

## When NOT to use

- Input is already a feature table or quantification table (e.g., MZmine2 output) rather than a sample-information metadata sheet
- The dataset files have not yet been uploaded to MassIVE, so filename matching against repository contents cannot be performed
- The goal is a follow-up targeted LC-MS quantitation workflow rather than metadata preparation for molecular networking submission

## Inputs

- Deposited MassIVE metadata file (tab-separated text, e.g., 3DMouse_duodenum_metadata.txt for MSV000083437)
- List of data file names uploaded to MassIVE (e.g., converted .mzXML/.mzML/.mgf open-format files)

## Outputs

- Conformance report (TSV) listing each row/column schema violation
- Normalized metadata table (tab-separated text) satisfying the GNPS schema with 'filename' first column and 'ATTRIBUTE_'-prefixed columns

## How to apply

Load the metadata file (e.g., 3DMouse_duodenum_metadata.txt from MSV000083437) as a tab-separated text table with pandas. Verify the first column header is exactly 'filename' and that every 'filename' cell matches the names of the data files uploaded to MassIVE via FTP client or browser. Confirm all remaining column headers are prefixed with 'ATTRIBUTE_' and contain no forbidden special characters such as '@', '#', '!', or spaces (metadata fields should be whitespace-separated tokens). Flag each row/column violation in a conformance report and write a normalized TSV copy that satisfies the GNPS schema before job submission.

## Related tools

- **GNPS** (Platform whose metadata schema ('filename' first column, 'ATTRIBUTE_' column prefix) the table must satisfy before molecular networking job submission) — http://gnps.ucsd.edu
- **MassIVE** (Repository holding the uploaded LC-MS data files whose names must be matched by the metadata 'filename' column)

## Examples

```
import pandas as pd; df = pd.read_csv('3DMouse_duodenum_metadata.txt', sep='\t'); assert df.columns[0] == 'filename'; assert all(c.startswith('ATTRIBUTE_') for c in df.columns[1:])
```

## Evaluation signals

- First column header of the metadata table is exactly 'filename'
- Every 'filename' cell matches a data file name uploaded to MassIVE for the dataset (e.g., MSV000083437) — no orphans in either direction
- All non-filename column headers begin with the 'ATTRIBUTE_' prefix
- No metadata field contains '@', '#', '!', or spaces; fields are whitespace-separated tokens
- Normalized output is a tab-separated text file that loads cleanly in pandas with the expected column structure

## Limitations

- GNPS molecular networking is qualitative (MS2 scans only); the metadata enables networking and living-data annotation but does not add quantitative calibration
- Only ~5–6% of MS2 spectra in untargeted metabolomics end up annotated, so metadata linking to library matches (cosine score ≥ 0.7 with 6 matched peaks) still leaves most spectra as 'dark matter'
- Statistics were not performed for the tutorial protocol example, so metadata conformance alone does not validate downstream biological claims

## Evidence

- [intro] Collate and upload a sample information (metadata) table: [evidence span withheld — anchor status: not_found]
- [supplementary] MSV000083437 accession in GNPS-MassIVE: [evidence span withheld — anchor status: not_found]
