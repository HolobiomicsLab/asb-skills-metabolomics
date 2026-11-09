---
name: filename-convention-checking
description: Use when preparing to submit a molecular networking or feature-based
  molecular networking job to GNPS and a sample-information/metadata table (e.g.,
  3DMouse_duodenum_metadata.txt from MSV000083437) must be conformance-checked and
  normalized.
when_to_use_negative:
- The metadata table is already a feature abundance table (e.g., MZmine2 or MS-Dial
  quantification output) rather than a per-sample sample-information sheet
- The target platform is not GNPS/MassIVE and its workflow does not impose the 'filename'/'ATTRIBUTE_'
  header convention
- You need to clean raw LC-MS spectra (peak filtering, precursor window deletion)
  rather than validate tabular sample metadata
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Platform whose molecular networking workflow imposes the metadata schema being
    validated ('filename' first column, 'ATTRIBUTE_' prefix)
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Repository holding the uploaded dataset whose file names must match the metadata
    'filename' column values
- name: pandas
  role: Python library used to load and inspect the tab-separated metadata table for
    header and field validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:99805eb4950ef633afd0866f54d580de2b5358918d34930ef03ac6abc414f16a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPSDocumentation
  tools:
  - GNPS
  - MassIVE
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# filename-convention-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a sample-information (metadata) table against the GNPS molecular networking schema before job submission: the first column must be 'filename' matching uploaded MassIVE file names, all other columns must carry the 'ATTRIBUTE_' prefix, and fields must avoid special characters such as '@', '#', '!' and spaces.

## When to use

Use when preparing to submit a molecular networking or feature-based molecular networking job to GNPS and a sample-information/metadata table (e.g., 3DMouse_duodenum_metadata.txt from MSV000083437) must be conformance-checked and normalized.

## When NOT to use

- The metadata table is already a feature abundance table (e.g., MZmine2 or MS-Dial quantification output) rather than a per-sample sample-information sheet
- The target platform is not GNPS/MassIVE and its workflow does not impose the 'filename'/'ATTRIBUTE_' header convention
- You need to clean raw LC-MS spectra (peak filtering, precursor window deletion) rather than validate tabular sample metadata

## Inputs

- Sample-information (metadata) table as a tab-separated text file (e.g., 3DMouse_duodenum_metadata.txt from MSV000083437)
- List of data file names uploaded to MassIVE for the target dataset (.mzXML/.mzML/.mgf files)

## Outputs

- Conformance report (TSV) flagging each row/column schema violation
- Normalized metadata table (TSV) satisfying the GNPS schema ('filename' first column, 'ATTRIBUTE_'-prefixed headers, no forbidden characters)

## How to apply

Load the metadata file with pandas as a tab-separated text table, since GNPS expects TSV. Check that the first column header is exactly 'filename' and that every 'filename' cell matches the names of the data files uploaded to MassIVE for the dataset (e.g., MSV000083437). Verify that every remaining column header is prefixed with 'ATTRIBUTE_' and that field values contain no forbidden special characters ('@', '#', '!') or spaces — use whitespace-separated tokens instead. Flag each row/column violation in a conformance report, then write a normalized TSV copy that satisfies the schema so the table can be uploaded as standardized sample information alongside the .mzXML/.mzML/.mgf data files.

## Related tools

- **GNPS** (Platform whose molecular networking workflow imposes the metadata schema being validated ('filename' first column, 'ATTRIBUTE_' prefix)) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Repository holding the uploaded dataset whose file names must match the metadata 'filename' column values)
- **pandas** (Python library used to load and inspect the tab-separated metadata table for header and field validation)

## Examples

```
import pandas as pd; df = pd.read_csv('3DMouse_duodenum_metadata.txt', sep='\t'); assert df.columns[0] == 'filename'; assert all(c.startswith('ATTRIBUTE_') for c in df.columns[1:])
```

## Evaluation signals

- First column header of the loaded TSV is exactly 'filename'
- Every value in the 'filename' column matches a data file name uploaded to MassIVE for the dataset (no unmatched or missing files)
- All non-first column headers begin with the 'ATTRIBUTE_' prefix
- No metadata field contains '@', '#', '!' or spaces; fields use whitespace-separated tokens
- Normalized output is a valid tab-separated text file with row count unchanged from the input

## Limitations

- Special characters such as '@', '#', '!' and spaces in metadata fields cause schema failures and must be removed rather than escaped
- The check is schema-specific to GNPS molecular networking conventions and does not transfer to other metadata standards (e.g., ISA-Tab) without adaptation
- Correct metadata alone does not guarantee successful networking; workflow parameters (precursor ion window, 50 Da filter window, cosine cutoff 0.7 with 6 matched peaks) are validated separately

## Evidence

- [intro] Collate and upload a sample information (metadata) table: "reproducibly create publishable molecular networks using standardized sample information (metadata)"
- [intro] Upload data files to MassIVE: "(iii) uploading data to the MassIVE public repository"
- [supplementary] MSV000083437 dataset: "MSV000083437 (germ-free (GF) and speci fi c-pathogen-free (SPF) mice)"
- [supplementary] Metadata file naming: [evidence span withheld — anchor status: not_found]
