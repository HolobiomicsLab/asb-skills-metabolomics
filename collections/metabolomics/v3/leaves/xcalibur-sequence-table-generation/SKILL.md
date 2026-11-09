---
name: xcalibur-sequence-table-generation
description: Use when a plate-based (e.g. 384-well) metadata sheet of compounds must
  be converted into an Xcalibur sequence.csv for Orbitrap ID-X flow injection MSn
  acquisition.
when_to_use_negative:
- The metadata sheet has not been cleaned yet — run the metadata cleanup Prefect flow
  (metadata_cleanup_prefect.py) first so required columns and identifiers are valid
- The target instrument is not an Orbitrap ID-X controlled by Xcalibur — the sequence
  layout is explicitly set up for that instrument
- The data are already acquired RAW files needing conversion (use ThermoRawFileParser)
  rather than pre-acquisition sequence generation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Orbitrap ID-X
  role: Target instrument for which the Xcalibur sequence table is generated (flow
    injection MSn acquisition)
- name: Xcalibur
  role: Instrument software whose sequence layout defines the format of the output
    sequence.csv
- name: Prefect
  role: Orchestrates the surrounding metadata cleanup/deployment workflow that produces
    the input metadata sheet
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: pip
  role: Installs the Python requirements needed to run the sequence creation scripts
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:20a64ba576781ee86113063a0353ce1c696866817521435d16b2a7f844ac02ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Orbitrap ID-X
  - Prefect
  - Xcalibur
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# xcalibur-sequence-table-generation

## Summary

Generates Xcalibur-format acquisition sequence tables for Orbitrap ID-X flow injection MSn runs by converting a cleaned metadata sheet into ordered sequence rows with auto-generated file names (date, sample ID, method, polarity).

## When to use

Use when a plate-based (e.g. 384-well) metadata sheet of compounds must be converted into an Xcalibur sequence.csv for Orbitrap ID-X flow injection MSn acquisition.

## When NOT to use

- The metadata sheet has not been cleaned yet — run the metadata cleanup Prefect flow (metadata_cleanup_prefect.py) first so required columns and identifiers are valid
- The target instrument is not an Orbitrap ID-X controlled by Xcalibur — the sequence layout is explicitly set up for that instrument
- The data are already acquired RAW files needing conversion (use ThermoRawFileParser) rather than pre-acquisition sequence generation

## Inputs

- Cleaned metadata sheet (e.g. CSV/Excel) with columns: unique sample id, plate_id (batch identifier), well_location (vial or well position)
- Instrument method file name(s) for the Orbitrap ID-X flow injection MSn runs
- Acquisition date stamp

## Outputs

- sequence.csv in Xcalibur sequence layout, one row per well, ordered positive- then negative-ionization per plate, with file names following the date-sampleID-method-polarity convention

## How to apply

Load the cleaned metadata sheet with pandas, requiring the columns 'unique sample id', 'plate_id' (batch identifier), and 'well_location' (vial or well position), following the shared metadata template from the msn_tree_library repository. Validate that unique sample IDs are assigned per well/plate combination to avoid collisions across the batch; missing structure or name fields should have been resolved upstream (e.g. PubChem Name search). For each well, auto-generate the acquisition file name composed of date, sample ID, method, and polarity. Sort wells within each plate so all positive-ionization-mode entries precede negative-ionization-mode entries, then write the ordered rows into an Xcalibur-format sequence table compatible with Orbitrap ID-X import and save it as sequence.csv.

## Related tools

- **Orbitrap ID-X** (Target instrument for which the Xcalibur sequence table is generated (flow injection MSn acquisition))
- **Xcalibur** (Instrument software whose sequence layout defines the format of the output sequence.csv)
- **Prefect** (Orchestrates the surrounding metadata cleanup/deployment workflow that produces the input metadata sheet) — https://github.com/corinnabrungs/msn_tree_library
- **pip** (Installs the Python requirements needed to run the sequence creation scripts) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
pip install requirements; python sequence_creation.py  # builds sequence.csv (Xcalibur layout, pos-then-neg per plate) from the cleaned metadata sheet
```

## Evaluation signals

- sequence.csv contains exactly one row per well of the plate(s) in the metadata sheet
- All required Xcalibur sequence columns are populated for every row
- Every sample ID is unique per well/plate combination with no collisions across the batch
- File names match the date-sampleID-method-polarity convention
- Rows are ordered within each plate_id with all positive-mode entries before negative-mode entries

## Limitations

- The sequence creation is set up specifically for the Orbitrap ID-X (Xcalibur); other instruments/control software need a different sequence layout
- The metadata sheet must follow the shared template with the same column names and minimum needed information, or required columns (unique sample id, plate_id, well_location) will be missing
- Some upstream database queries need a local file and/or special access, otherwise they must be set to False in jobs.py, which affects metadata completeness before sequencing

## Evidence

- [methods] sequence creation target instrument: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] metadata template requirement: "Please use the
[template](https://docs.google.com/spreadsheets/d/1v6_IlGS3VgycGc-mSSdNeocY-CFXpONVZbuh3XNLX2E/edit?usp=sharing)
for your metadata for having same column names and minimum needed"
- [readme] PubChem name fallback: "If no
structure information is provided, it is queried from PubChem by Name search."
- [methods] requirements install: "## Requirements for running:

```bash
pip install requirements
```"
