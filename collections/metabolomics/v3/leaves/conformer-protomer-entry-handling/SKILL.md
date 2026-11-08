---
name: conformer-protomer-entry-handling
description: Use when building LC-IM-MS (drift tube) database entries where one molecular
  formula yields chromatographically separated isomers or multiple IM-separated ion
  species (protomers/conformers) that must each become its own database entry.
when_to_use_negative:
- The input is not drift tube IM-MS data (CCSfind is built for LC-IM-MS drift tube
  IM-MS measurements; trapping-TIMS/TWIMS-style workflows are out of scope)
- Only summary CCS values are needed and the full IM spectra / multiple ion species
  per compound should not be preserved as separate entries
- The mzML file has not been processed and manually reviewed via the isotopologue-matching-confirmed
  peak picking step, so entries would lack reviewed EICs and DTCCS N2 annotations
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Handles chromatographically separated isomers and IM separation of isomeric
    ions (protomers/conformers) with user control over SQL-format database entry output
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Loads the reviewed CCSfind results for assembling per-species compound records
    before database commitment
- name: pandas
  role: Alternative data frame backend for loading reviewed peak/feature data when
    assembling entries
- name: Python 3.9
  role: Runtime environment (conda env with python 3.9) in which CCSfind processing
    and database-entry assembly are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:262c88f44d44dc13cebcb9e50955da4ce01f83110bf51249ea844ea204b4c3de
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
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# conformer-protomer-entry-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for encoding chromatographically separated isomers and IM-separated protomers/conformers of the same input molecular formula as distinct SQL-format database entries, while retaining the full ion mobility spectra (CCS distributions) rather than only summary values.

## When to use

Use when building LC-IM-MS (drift tube) database entries where one molecular formula yields chromatographically separated isomers or multiple IM-separated ion species (protomers/conformers) that must each become its own database entry.

## When NOT to use

- The input is not drift tube IM-MS data (CCSfind is built for LC-IM-MS drift tube IM-MS measurements; trapping-TIMS/TWIMS-style workflows are out of scope)
- Only summary CCS values are needed and the full IM spectra / multiple ion species per compound should not be preserved as separate entries
- The mzML file has not been processed and manually reviewed via the isotopologue-matching-confirmed peak picking step, so entries would lack reviewed EICs and DTCCS N2 annotations

## Inputs

- Profile-mode mzML file from LC-DTIM-MS measurements
- List of molecular formulae to search in the mzML file
- Reviewed CCSfind results (EICs, full IM spectra, DTCCS N2 values) exported/loaded via polars or pandas
- Single-field CCS calibration coefficients (Tfix and Beta)
- Identity of the drift gas (e.g., nitrogen)
- Primary and additional adduct ion species (positive/negative mode)

## Outputs

- SQL-format database entries, one per isomer/protomer/conformer, each containing EICs and the full ion mobility spectrum
- Entries annotated with DTCCS N2 values and experimental settings
- Entries linked to PubChem CIDs for the input molecular formulae

## How to apply

After running CCSfind processing ('python processor.py -g config.ini') on the parsed mzML input, load the reviewed results with polars or pandas and assemble one compound record per detected species. For each record, retain the extracted ion chromatograms (EICs) and the full normalized ion mobility spectra inside the database entry, alongside the DTCCS N2 value and the experimental settings recorded during manual review. Encode each chromatographically separated isomer and each IM-separated protomer or conformer of the same ion species as a separate entry, using CCSfind's simple user control over the output for new database entries in SQL format. Link each input molecular formula to its PubChem CID so entries are chemically resolvable. The rationale is that existing analytical databases do not transfer the maximum IM-MS information — full mobility spectra (CCS distributions) and multiple ion species per compound — so the skill packages the full retrievable dataset per entry at a moderate cost of approximately 200 KB of database size per entry.

## Related tools

- **CCSfind** (Handles chromatographically separated isomers and IM separation of isomeric ions (protomers/conformers) with user control over SQL-format database entry output) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Loads the reviewed CCSfind results for assembling per-species compound records before database commitment)
- **pandas** (Alternative data frame backend for loading reviewed peak/feature data when assembling entries)
- **Python 3.9** (Runtime environment (conda env with python 3.9) in which CCSfind processing and database-entry assembly are executed)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Each detected protomer/conformer and chromatographically separated isomer of the same formula appears as its own SQL entry rather than being merged into one compound record
- The full IM spectrum (CCS distribution) for each entry is retrievable from the committed database, not just a single CCS value
- Every entry carries a DTCCS N2 value, drift gas identity, and experimental settings captured during manual review
- Each entry is linked to the correct PubChem CID for its input molecular formula
- Committing each reviewed compound entry increases the database file size by the reported moderate ~200 KB per entry

## Limitations

- Requires drift tube IM-MS (LC-DTIM-MS) profile mzML input; performance claims were validated on desktop hardware with 32 GB RAM
- Retention of full ion mobility spectra inflates the database by ~200 KB per entry, which may be substantial for large compound libraries
- Output control for new database entries must be configured by the user; misconfigured output could merge isomeric species into a single entry
- Additional ion species support depends on calibration coefficients (Tfix, Beta) and correct drift gas identity being supplied

## Evidence
