---
name: lcms-injection-list-formatting
description: Use when preparing flow-injection MSn acquisition for a Thermo Orbitrap
  ID-X and you have plate-based metadata (unique sample id, plate_id, well_location)
  that must become an Xcalibur import sequence.
when_to_use_negative:
- The metadata sheet has not yet passed the metadata cleanup flow (missing or non-unique
  sample ids, inconsistent column names) — run metadata_cleanup_prefect.py first
- The target instrument is not a Thermo Orbitrap ID-X running Xcalibur, since the
  sequence table layout is instrument-specific
- You need LC-MS rather than flow-injection sequences where ordering per plate by
  polarity does not apply
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Orbitrap ID-X
  role: Target instrument whose import format the sequence table must match
- name: Xcalibur
  role: Acquisition software that imports the generated sequence.csv
- name: Prefect
  role: Orchestrates the pipeline steps (metadata cleanup and downstream flows) surrounding
    sequence creation
  repo: corinnabrungs/msn_tree_library
- name: pip
  role: Installs the pipeline requirements needed to run the scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:37b8631be3995b95d44e93ebe768e60b32b1d1ad1230cd20392b35dcc3731737
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Orbitrap ID-X
  - Prefect
  - Xcalibur
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lcms-injection-list-formatting

## Summary

Formats a cleaned metadata sheet into an Xcalibur-compatible acquisition sequence (sequence.csv) for the Orbitrap ID-X, with auto-generated file names (date, sample ID, method, polarity) and wells ordered positive- then negative-ionization per plate.

## When to use

Use when preparing flow-injection MSn acquisition for a Thermo Orbitrap ID-X and you have plate-based metadata (unique sample id, plate_id, well_location) that must become an Xcalibur import sequence.

## When NOT to use

- The metadata sheet has not yet passed the metadata cleanup flow (missing or non-unique sample ids, inconsistent column names) — run metadata_cleanup_prefect.py first
- The target instrument is not a Thermo Orbitrap ID-X running Xcalibur, since the sequence table layout is instrument-specific
- You need LC-MS rather than flow-injection sequences where ordering per plate by polarity does not apply

## Inputs

- Cleaned metadata sheet (e.g. CSV/Excel) with columns: unique sample id, plate_id (batch identifier), well_location (vial or well position)
- Acquisition method name(s) for the flow-injection MSn runs
- Acquisition date and ionization polarity labels (positive/negative)

## Outputs

- sequence.csv — one row per well in the Xcalibur sequence table layout, compatible with Orbitrap ID-X import, with date-sampleID-method-polarity file names ordered positive then negative per plate

## How to apply

Load the cleaned metadata sheet with pandas, requiring the columns 'unique sample id', 'plate_id' (batch identifier), and 'well_location' (vial or well position). Validate that sample IDs are unique per well/plate combination so no collisions occur across the batch. Auto-generate each acquisition file name from the date, unique sample identifier, method, and polarity. Sort the wells so that within each plate_id all positive-ionization-mode entries precede negative-ionization entries, then write the ordered rows into the Xcalibur sequence table layout and save as sequence.csv. The Orbitrap ID-X/Xcalibur pairing is fixed by the pipeline design — the sequence creation 'is setup for the Orbitrap ID-X (Xcalibur)'.

## Related tools

- **Orbitrap ID-X** (Target instrument whose import format the sequence table must match)
- **Xcalibur** (Acquisition software that imports the generated sequence.csv)
- **Prefect** (Orchestrates the pipeline steps (metadata cleanup and downstream flows) surrounding sequence creation) — corinnabrungs/msn_tree_library
- **pip** (Installs the pipeline requirements needed to run the scripts)

## Evaluation signals

- sequence.csv contains exactly one row per well in the metadata sheet
- All required Xcalibur sequence columns are populated for every row
- Sample IDs are unique across the well/plate combination with no collisions in the batch
- File names follow the date-sampleID-method-polarity convention
- Within each plate_id, all positive-mode entries appear before negative-mode entries

## Limitations

- Requires the metadata template with shared column names and minimum needed information; deviation breaks the query
- Sequence layout is specific to Orbitrap ID-X / Xcalibur and does not generalize to other instruments
- Sample-ID collisions across plate_id/well_location combinations will corrupt the sequence if not validated

## Evidence

- [methods] Sequence creation setup for Orbitrap ID-X/Xcalibur with required metadata columns: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] Metadata must follow the shared template with required column names: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
