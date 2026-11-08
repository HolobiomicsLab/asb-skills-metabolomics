---
name: sql-database-entry-construction
description: Use when manually reviewed LC-IM-MS (drift tube) results must be committed
  to a searchable database in which the full mobility spectrum and per-compound metadata
  can be retrieved, not just summary CCS values.
when_to_use_negative:
- Input is only a summary feature/peak table without the full ion mobility spectra
  — the skill's purpose is retaining full mobility spectra in the database
- Data were not acquired with drift tube IM-MS (CCSfind targets drift tube IM-MS data
  of small molecules)
- The analysis has not passed isotopologue matching confirmation and manual review
  — entries should only be committed after annotation with experimental settings
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: CCSfind
  role: Generates the reviewed results (EICs, IM spectra, DTCCS N2 values) and writes
    new database entries in SQL format with user control over output
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Load and assemble reviewed CCSfind result records before committing them to
    the database
- name: pandas
  role: Alternative to polars for loading reviewed results and assembling compound
    records
- name: numpy
  role: Dependency installed alongside polars/pandas for handling spectral arrays
    in the result data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:6affd9f1fed3d82ba519d70a3459bbc5c98f59e5976f9e18a26a038b0afc5a25
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - conda
  - numpy
  - pandas
  - pip
  - polars
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sql-database-entry-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds SQL-format database entries from reviewed CCSfind results that retain the full ion mobility spectra (CCS distributions), EICs, DTCCS N2 values, experimental settings, and PubChem CIDs, at a reported storage cost of ~200 KB per entry.

## When to use

Use when manually reviewed LC-IM-MS (drift tube) results must be committed to a searchable database in which the full mobility spectrum and per-compound metadata can be retrieved, not just summary CCS values.

## When NOT to use

- Input is only a summary feature/peak table without the full ion mobility spectra — the skill's purpose is retaining full mobility spectra in the database
- Data were not acquired with drift tube IM-MS (CCSfind targets drift tube IM-MS data of small molecules)
- The analysis has not passed isotopologue matching confirmation and manual review — entries should only be committed after annotation with experimental settings

## Inputs

- Reviewed CCSfind results (peak-picked features after isotopologue matching confirmation) from profile mzML LC-DTIM-MS files
- List of input molecular formulae with predicted ion species
- Single-field CCS calibration coefficients (Tfix and Beta) and drift gas identity (e.g., nitrogen)
- Manual review annotations (experimental settings, PubChem CID per formula)

## Outputs

- SQL-format database entries containing EICs, full IM spectra, DTCCS N2 values, experimental settings, and PubChem CIDs
- Separate entries for chromatographically separated isomers and IM-separated protomers/conformers
- Database file growing by approximately 200 KB per committed entry

## How to apply

After running the analysis via 'python processor.py -g config.ini' on the parsed mzML input, load the reviewed CCSfind results with polars or pandas. Assemble each compound record so that the extracted ion chromatograms (EICs) and full normalized IM spectra are embedded in the database entry rather than only summary DTCCS N2 values. Annotate each entry with its DTCCS N2 value and the experimental settings recorded during the manual review step, and link the PubChem CID for each input molecular formula. Encode chromatographically separated isomers and IM-separated protomers/conformers of the same ion species as separate entries, under simple user control over the output, then commit the assembled records as SQL-format database entries from which the full dataset can be retrieved.

## Related tools

- **CCSfind** (Generates the reviewed results (EICs, IM spectra, DTCCS N2 values) and writes new database entries in SQL format with user control over output) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Load and assemble reviewed CCSfind result records before committing them to the database)
- **pandas** (Alternative to polars for loading reviewed results and assembling compound records)
- **numpy** (Dependency installed alongside polars/pandas for handling spectral arrays in the result data)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Each committed compound entry increases the database file size by the reported moderate ~200 KB per entry
- Full ion mobility spectra (CCS distributions) are retrievable from each database entry, not only summary DTCCS N2 values
- Every entry carries its DTCCS N2 value, the experimental settings recorded at manual review, and a PubChem CID link for the input formula
- Chromatographically separated isomers and IM-separated protomers/conformers appear as distinct database entries as configured by the user

## Limitations

- Database entries incur a moderate ~200 KB size increase each because full IM spectra are stored, so very large libraries grow quickly
- Only drift tube IM-MS (LC-DTIM-MS) data with single-field CCS calibration coefficients (Tfix, Beta) and a known drift gas (e.g., nitrogen) are supported
- Only reviewed, annotated results should be committed; entries are committed after the manual review and annotation step, not directly from raw processing

## Evidence
