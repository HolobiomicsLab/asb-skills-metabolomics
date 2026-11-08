---
name: im-spectra-database-encoding
description: Use when committing manually reviewed drift tube LC-IM-MS results into
  a searchable analytical database and you need full mobility spectra (CCS distributions)
  retained per compound rather than only summary CCS values.
when_to_use_negative:
- Your analytical database only needs to store summary CCS values, not the full mobility
  spectra (CCS distributions) per compound
- Input is not drift tube IM-MS (LC-DTIM-MS) profile mzML data with known CCS calibration
  coefficients and drift gas identity
edam_operation: http://edamontology.org/operation_3431
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSfind
  role: Primary tool whose output stage assembles reviewed results (EICs, full IM
    spectra, DTCCS N2, experimental settings, PubChem CID) into SQL-format database
    entries with user control over output
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Dataframe library for loading and assembling reviewed CCSfind results before
    database commit
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: Alternative to polars for loading curated peak and feature data when assembling
    compound records
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Runtime environment (installed via conda/pip) in which CCSfind processing
    and database entry generation are executed
  repo: https://github.com/sangeeta97/ccs_find.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:aa42a1e4e043e3c21afc87a264fc2aad6b0e0522e2affed76a49184e3f4c7d59
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
  - Python 3.9
  - conda
  - numpy
  - pandas
  - pip
  - polars
  edam_operation: http://edamontology.org/operation_3431
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# im-spectra-database-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes reviewed CCSfind results—including EICs, full normalized ion mobility spectra, DTCCS N2 values, experimental settings, and PubChem CID links—into SQL-format database entries so the full IM-MS dataset remains retrievable, at a reported storage cost of ~200 KB per entry.

## When to use

Use when committing manually reviewed drift tube LC-IM-MS results into a searchable analytical database and you need full mobility spectra (CCS distributions) retained per compound rather than only summary CCS values.

## When NOT to use

- Your analytical database only needs to store summary CCS values, not the full mobility spectra (CCS distributions) per compound
- Input is not drift tube IM-MS (LC-DTIM-MS) profile mzML data with known CCS calibration coefficients and drift gas identity

## Inputs

- Reviewed CCSfind results from LC-DTIM-MS mzML processing (parsed via processor.py with config.ini)
- DTCCS N2 values and experimental settings from the manual review step
- Input molecular formulae with PubChem CID links
- Single-field CCS calibration coefficients (Tfix, Beta) and drift gas identity (e.g., nitrogen)

## Outputs

- SQL-format database entries containing EICs, full IM spectra, DTCCS N2 values, experimental settings, and PubChem CIDs
- Separate database entries for chromatographically separated isomers and IM-separated protomers/conformers

## How to apply

After running 'python processor.py -g config.ini' on the parsed mzML input, load the reviewed CCSfind results with polars or pandas. Assemble each compound record so extracted ion chromatograms and the full normalized IM spectra are stored inside the entry, not just the DTCCS N2 summary value. Annotate each entry with its DTCCS N2 value and the experimental settings recorded during manual review, and add the PubChem CID link for the input molecular formula. Encode chromatographically separated isomers and IM-separated protomers/conformers as separate entries, with user control over which new SQL-format entries are output; commit the records so the full dataset can be retrieved.

## Related tools

- **CCSfind** (Primary tool whose output stage assembles reviewed results (EICs, full IM spectra, DTCCS N2, experimental settings, PubChem CID) into SQL-format database entries with user control over output) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Dataframe library for loading and assembling reviewed CCSfind results before database commit) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (Alternative to polars for loading curated peak and feature data when assembling compound records) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Runtime environment (installed via conda/pip) in which CCSfind processing and database entry generation are executed) — https://github.com/sangeeta97/ccs_find.git

## Examples

```
conda create --name ccs_find python=3.9; conda activate ccs_find; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- Each committed compound entry increases the database file size by the reported moderate ~200 KB, confirming full IM spectra (not just summary values) are stored
- The full dataset (EICs and full normalized IM spectra) can be retrieved from each SQL-format entry after commit
- Chromatographically separated isomers and IM-separated protomers/conformers appear as separate database entries as configured by the user
- Each entry contains the DTCCS N2 value, the experimental settings from manual review, and the correct PubChem CID for the input molecular formula

## Limitations

- Storing full ion mobility spectra causes a moderate increase in database size of approximately 200 KB per entry, which may be substantial for very large compound sets
- Python 3.9 is required for the CCSfind environment; the tool is designed for drift tube IM-MS data with single-field CCS calibration coefficients
- The tool needs profile data mzML input from LC-DTIM-MS; processed/centroid-only input outside its workflow may not be supported

## Evidence

- [results] ~200 KB per entry storage cost: [evidence span withheld — anchor status: not_found]
