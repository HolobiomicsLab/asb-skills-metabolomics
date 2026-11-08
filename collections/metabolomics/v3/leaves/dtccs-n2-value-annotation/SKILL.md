---
name: dtccs-n2-value-annotation
description: Use when reviewed drift tube IM-MS results from CCSfind must be committed
  to a database with DTCCS N2 values, experimental settings, and full mobility spectra
  retrievable per entry.
when_to_use_negative:
- When the goal is only summary CCS values without retrievable full mobility spectra
  — CCSfind exists because current databases do not transfer the full mobility spectra
  (CCS distributions)
- When input data are not profile LC-DTIM-MS mzML files processed by CCSfind (the
  pipeline expects drift tube IM-MS data with Tfix/Beta calibration and drift gas
  identity)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Produces the reviewed results and provides user control over SQL-format database
    entry output; annotation and commit step is part of its workflow
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Load the reviewed CCSfind results after processing before assembling database
    entries
- name: pandas
  role: Alternative to polars for loading reviewed results prior to entry assembly
- name: Python 3.9
  role: Runtime environment for processor.py and the database assembly workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:a6d8c3dde2fd43d5473d86ce05f7b959c8368cd053ccc8334fe9acbc527a8b6f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dtccs-n2-value-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Packages manually reviewed CCSfind results into SQL-format database entries that retain extracted ion chromatograms, full normalized ion mobility spectra, annotated DTCCS N2 values and experimental settings, at a reported cost of roughly 200 KB of database size per compound entry.

## When to use

Use when reviewed drift tube IM-MS results from CCSfind must be committed to a database with DTCCS N2 values, experimental settings, and full mobility spectra retrievable per entry.

## When NOT to use

- When the goal is only summary CCS values without retrievable full mobility spectra — CCSfind exists because current databases do not transfer the full mobility spectra (CCS distributions)
- When input data are not profile LC-DTIM-MS mzML files processed by CCSfind (the pipeline expects drift tube IM-MS data with Tfix/Beta calibration and drift gas identity)

## Inputs

- Reviewed CCSfind results (EICs and full normalized IM spectra) from processed LC-DTIM-MS mzML files
- DTCCS N2 values computed with the single-field CCS calibration coefficients (Tfix and Beta) and the drift gas identity (e.g., nitrogen)
- Experimental settings recorded during the manual review step
- Input molecular formulae for PubChem CID linking

## Outputs

- SQL-format database entries per compound, each retaining EICs, full IM spectra, DTCCS N2 value, experimental settings, and PubChem CID
- Separate entries for chromatographically separated isomers and IM-separated protomers/conformers
- Database file growing by approximately 200 KB per committed entry

## How to apply

After running 'python processor.py -g config.ini' on the parsed mzML input, load the reviewed CCSfind results with polars or pandas. Assemble each compound record so that the extracted ion chromatograms (EICs) and full normalized IM spectra are stored inside the database entry, not only summary CCS values. Annotate every entry with its DTCCS N2 value and the experimental settings captured during the manual review step, and add the PubChem CID link for each input molecular formula. Encode chromatographically separated isomers and IM-separated protomers or conformers as separate entries, using CCSfind's user control over SQL-format database entry output. Validate by checking that each committed entry increases the database file size by the reported moderate ~200 KB and that the full dataset remains retrievable.

## Related tools

- **CCSfind** (Produces the reviewed results and provides user control over SQL-format database entry output; annotation and commit step is part of its workflow) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Load the reviewed CCSfind results after processing before assembling database entries)
- **pandas** (Alternative to polars for loading reviewed results prior to entry assembly)
- **Python 3.9** (Runtime environment for processor.py and the database assembly workflow)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Each committed compound entry increases the database file size by approximately 200 KB, matching the reported moderate increase per entry
- Every entry contains the full normalized IM spectrum and EIC, not just the DTCCS N2 summary value
- Each entry carries an annotated DTCCS N2 value plus the experimental settings from the manual review step
- PubChem CID is present for each input molecular formula, and isomers/protomers/conformers appear as separate entries
- A query on the committed SQL database retrieves the full dataset for an entry

## Limitations

- Requires profile data mzML input from LC-DTIM-MS plus the 5 compulsory inputs (molecular formulae list, Tfix and Beta calibration coefficients, drift gas identity, primary ion species); missing any of these blocks entry generation
- Python 3.9 is a must for the CCSfind environment, so older/newer Python versions are not supported out of the box
- Committing full IM spectra inflates database size (~200 KB per entry), which is a deliberate trade-off over summary-only databases

## Evidence
