---
name: gcms-library-hit-table-parsing
description: Use when a GC-MS vendor output (e.g. Agilent Unknowns Analysis hit table)
  must be converted into a uafR-compatible data frame with exact column names for
  downstream spreadOut()/mzExacto() processing.
when_to_use_negative:
- Input is already a uafR spread database (spread_database.rds) — no re-parsing needed
- Data comes from LC-MS, MS2/MS3, or ion mobility workflows — uafR has not been tested
  in those contexts
- The table is a chemical library or categorate() output rather than a vendor peak
  table
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: uafR
  role: Package that consumes the parsed hit table via spreadOut() and mzExacto()
    for automated GC-MS peak-area retrieval
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime for read.csv-based parsing and schema validation before spreading
- name: devtools
  role: Installs uafR directly from GitHub so the parsed table can be piped into spreadOut()
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:073bfccd3c0a3ca87fcb3facdcc6286f5434b1b46240619b995bc5d846fdcf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - LOTUS
  - R
  - RStudio
  - devtools
  - uafR
  - webchem
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gcms-library-hit-table-parsing

## Summary

Parses an Agilent Unknowns Analysis GC-MS library-hit CSV into the strict column schema that uafR expects (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), validating duplicates and required fields before any peak-area retrieval.

## When to use

Use when a GC-MS vendor output (e.g. Agilent Unknowns Analysis hit table) must be converted into a uafR-compatible data frame with exact column names for downstream spreadOut()/mzExacto() processing.

## When NOT to use

- Input is already a uafR spread database (spread_database.rds) — no re-parsing needed
- Data comes from LC-MS, MS2/MS3, or ion mobility workflows — uafR has not been tested in those contexts
- The table is a chemical library or categorate() output rather than a vendor peak table

## Inputs

- Vendor GC-MS library-hit .csv export (e.g. Agilent Unknowns Analysis) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- R data frame read via read.csv(check.names = FALSE)

## Outputs

- Validated data frame with the exact six column names in no particular order
- spreadOut()-ready peak table expanded to one record per datapoint with unique identifiers

## How to apply

Load the vendor-exported .csv with read.csv(..., check.names = FALSE) so that column names are not altered. Verify that all six required columns — 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor' — are present, in any order; stop with stopifnot() if any are missing. Reject duplicate peak keys as documented in the spreadOut() reconstruction (task_004), and confirm that Match.Factor is numeric for threshold filtering later. The parsed table is then handed directly to spreadOut() which expands each row into a large database with unique IDs and fetches per-compound PubChem CIDs via webchem's get_cid(); compounds without a CID fall back to NCI CACTUS for canonical SMILES.

## Related tools

- **uafR** (Package that consumes the parsed hit table via spreadOut() and mzExacto() for automated GC-MS peak-area retrieval) — https://github.com/castratton/uafR
- **R** (Runtime for read.csv-based parsing and schema validation before spreading)
- **devtools** (Installs uafR directly from GitHub so the parsed table can be piped into spreadOut())

## Examples

```
raw_gcms <- read.csv("gcms-export.csv", check.names = FALSE); stopifnot(all(c("Component.RT","Component.Area","Base.Peak.MZ","File.Name","Compound.Name","Match.Factor") %in% names(raw_gcms))); spread <- spreadOut(raw_gcms)
```

## Evaluation signals

- stopifnot(all(required_columns %in% names(raw_gcms))) returns TRUE for the six required column names
- No duplicate peak keys remain after validation (spreadOut() rejects them)
- Match.Factor values are numeric and comparable (e.g. > 80 used for top-hit subsetting in the README example)
- Parsed table row count matches the vendor export row count before spreadOut() expansion

## Limitations

- The column names are a strict requirement — renaming them (e.g. via read.csv default check.names) breaks spreadOut()
- uafR has not been tested with Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS data
- Algorithms do not apply to MS2, MS3, or ion mobility + MS2 data until those spectra are published in PubChem

## Evidence

- [readme] The column names MUST include: 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor' in no particular order.: [evidence span withheld — anchor status: external_doc]
- [readme] Read vendor output without changing its column names: [evidence span withheld — anchor status: external_doc]
- [methods] spreadOut() expands every recorded data point into a large database with unique IDs, and fetches per-compound annotations: [evidence span withheld — anchor status: not_found]
- [methods] Reject duplicate peak keys and expand the table into one record per datapoint with unique identifiers: [evidence span withheld — anchor status: not_found]
