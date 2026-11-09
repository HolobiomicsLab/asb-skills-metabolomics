---
name: ionization-mode-ordering-per-plate
description: Use when converting a cleaned metadata sheet (unique sample id, plate_id,
  well_location) into an Xcalibur sequence table for Orbitrap ID-X flow-injection
  MSn, and wells must be ordered positive-mode first, then negative-mode, per plate.
when_to_use_negative:
- The target instrument is not the Orbitrap ID-X / Xcalibur sequence format — the
  sequence creation is explicitly 'setup for the Orbitrap ID-X (Xcalibur)'.
- The metadata has not been cleaned yet (missing unique sample id, plate_id, or well_location
  columns); run the metadata_cleanup_prefect flow first.
tools:
- name: Xcalibur
  role: Target sequence-table format; generated sequence.csv is compatible with Xcalibur/Orbitrap
    ID-X import
- name: Orbitrap ID-X
  role: Instrument for which the acquisition sequence is generated
- name: pandas
  role: Used to load and sort the cleaned metadata sheet before writing the sequence
    rows
- name: Prefect
  role: Orchestrates the surrounding workflow (metadata cleanup flow) that feeds the
    sequence creation step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:3341ade08c8fb4b2fb7a4d8d7379adc2b2d0f6b6fdf73fc981bd04fff5bb9014
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Orbitrap ID-X
  - Prefect
  - Xcalibur
  - pandas
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ionization-mode-ordering-per-plate

## Summary

Orders acquisition wells so that, within each plate_id (batch), all positive-ionization entries precede negative-ionization entries when building an Xcalibur sequence for the Orbitrap ID-X. This keeps flow-injection MSn runs batched per polarity per plate, avoiding repeated instrument-mode switching mid-plate.

## When to use

Use when converting a cleaned metadata sheet (unique sample id, plate_id, well_location) into an Xcalibur sequence table for Orbitrap ID-X flow-injection MSn, and wells must be ordered positive-mode first, then negative-mode, per plate.

## When NOT to use

- The target instrument is not the Orbitrap ID-X / Xcalibur sequence format — the sequence creation is explicitly 'setup for the Orbitrap ID-X (Xcalibur)'.
- The metadata has not been cleaned yet (missing unique sample id, plate_id, or well_location columns); run the metadata_cleanup_prefect flow first.

## Inputs

- cleaned metadata sheet (e.g. xlsx/csv) with columns: unique sample id, plate_id (batch/plate name), well_location (vial or well position)
- acquisition method name(s) for the flow-injection MSn runs
- ionization polarity assignments (positive/negative) per well

## Outputs

- sequence.csv in Xcalibur sequence-table layout (Orbitrap ID-X importable), one row per well, ordered positive-then-negative per plate
- per-well acquisition file names following the date-sampleID-method-polarity convention

## How to apply

Load the cleaned metadata sheet with pandas, requiring the columns 'unique sample id', 'plate_id' (the batch/plate identifier), and 'well_location' (vial or well position) as stated in the README. Validate that each well/plate combination gets a unique sample ID so there are no collisions across the batch. Auto-generate each acquisition file name from date, sample ID, method, and polarity following the date-sampleID-method-polarity convention. Sort rows so that within each plate_id all positive-ionization-mode entries come before negative-ionization-mode entries, then write the ordered rows into the Xcalibur sequence layout (Orbitrap ID-X-compatible) and save as sequence.csv.

## Related tools

- **Xcalibur** (Target sequence-table format; generated sequence.csv is compatible with Xcalibur/Orbitrap ID-X import)
- **Orbitrap ID-X** (Instrument for which the acquisition sequence is generated)
- **pandas** (Used to load and sort the cleaned metadata sheet before writing the sequence rows)
- **Prefect** (Orchestrates the surrounding workflow (metadata cleanup flow) that feeds the sequence creation step)

## Evaluation signals

- sequence.csv contains exactly one row per well of the input metadata sheet, with no dropped or duplicated wells.
- All unique sample IDs are distinct across the batch (no collisions between well/plate combinations).
- Every generated file name matches the date-sampleID-method-polarity convention.
- Within each plate_id, all positive-ionization rows appear contiguously before the negative-ionization rows.
- All required Xcalibur sequence columns are populated (no empty cells) so the file imports into Orbitrap ID-X.

## Limitations

- The output is specific to the Xcalibur/Orbitrap ID-X sequence layout; it does not produce sequences for other instrument control software.
- Metadata must follow the shared template with the same column names and minimum required information; non-conforming sheets will fail the required-column load.
- Some external database lookups need a local file and/or special access and must otherwise be set to False in jobs.py (upstream of sequence creation).

## Evidence

- [methods] Sequence creation is set up for Orbitrap ID-X with required metadata columns: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] Metadata must follow the shared template with fixed column names: "Please use the
[template] for your metadata for having same column names and minimum needed information for the query."
- [intro] PubChem Name search fills missing structure information during metadata preparation: "If no structure information is provided, it is queried from PubChem by Name search."
