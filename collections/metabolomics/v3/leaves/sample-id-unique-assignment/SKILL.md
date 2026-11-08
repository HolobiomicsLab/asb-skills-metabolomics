---
name: sample-id-unique-assignment
description: Use when generating acquisition sequence files from a plate-based metadata
  sheet where multiple wells or plates could collide on identifiers, and downstream
  Xcalibur file names must be unambiguous.
when_to_use_negative:
- Metadata comes from a non-plate, single-sample submission without plate_id/well_location
  structure — the well/plate collision logic does not apply
- The metadata sheet has not yet been cleaned via the metadata_cleanup_prefect flow
  (missing required columns or structure info)
- The target instrument is not Xcalibur/Orbitrap ID-X based — the sequence layout
  and file naming convention are instrument-specific
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Xcalibur
  role: Target sequence-table format and import system for the generated acquisition
    sequence on the Orbitrap ID-X
- name: Orbitrap ID-X
  role: Instrument whose flow injection MSn acquisition the unique sample IDs and
    file names must serve
- name: Prefect
  role: Orchestrates the metadata cleanup flow that produces the validated metadata
    sheet consumed by sequence creation
  repo: corinnabrungs/msn_tree_library
- name: pip
  role: Installs the Python requirements for running the sequence-creation scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:40eddbeaab1ac9549edbeeb066d703dcdec1cc97f88a1a617157e8e155b325bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Orbitrap ID-X
  - Prefect
  - Xcalibur
  - pip
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sample-id-unique-assignment

## Summary

Assigns and validates unique sample identifiers per well/plate combination so that each acquisition row in an Xcalibur sequence table for flow injection MSn on the Orbitrap ID-X maps to exactly one well with no filename collisions across the batch.

## When to use

Use when generating acquisition sequence files from a plate-based metadata sheet where multiple wells or plates could collide on identifiers, and downstream Xcalibur file names must be unambiguous.

## When NOT to use

- Metadata comes from a non-plate, single-sample submission without plate_id/well_location structure — the well/plate collision logic does not apply
- The metadata sheet has not yet been cleaned via the metadata_cleanup_prefect flow (missing required columns or structure info)
- The target instrument is not Xcalibur/Orbitrap ID-X based — the sequence layout and file naming convention are instrument-specific

## Inputs

- Cleaned metadata sheet (pandas-readable, e.g. CSV/Excel) with columns 'unique sample id', 'plate_id' (batch identifier), and 'well_location' (vial or well position)
- Acquisition method name and ionization polarity settings for the Orbitrap ID-X flow injection run

## Outputs

- Per-well rows with validated, non-colliding unique sample IDs
- Xcalibur-format sequence table (sequence.csv) with one row per well, each carrying a unique date-sampleID-method-polarity file name, ordered positive then negative per plate

## How to apply

Load the cleaned metadata sheet with pandas, requiring the columns unique sample id, plate_id (the batch/plate name), and well_location (vial or well position) as specified in the msn_tree_library README. For each well/plate combination, validate that the sample ID is present and assign a unique identifier so that IDs never collide across plates within the batch. Use this unique ID when composing the acquisition file name from the date, sample ID, method, and polarity in the Xcalibur sequence layout, so every generated RAW file name is unambiguous. Order the rows within each plate_id with all positive-ionization entries before negative-ionization entries, then write the sequence table for Orbitrap ID-X import.

## Related tools

- **Xcalibur** (Target sequence-table format and import system for the generated acquisition sequence on the Orbitrap ID-X)
- **Orbitrap ID-X** (Instrument whose flow injection MSn acquisition the unique sample IDs and file names must serve)
- **Prefect** (Orchestrates the metadata cleanup flow that produces the validated metadata sheet consumed by sequence creation) — corinnabrungs/msn_tree_library
- **pip** (Installs the Python requirements for running the sequence-creation scripts)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- sequence.csv contains exactly one row per well with all required Xcalibur columns populated
- All unique sample IDs are distinct across the batch, with no collisions between wells or plates
- Every generated file name follows the date-sampleID-method-polarity convention
- Within each plate_id, all positive-ionization rows precede the negative-ionization rows
- Each row's well_location corresponds to a valid vial or well position on the stated plate

## Limitations

- The metadata template and column names are fixed — sheets must follow the shared template with the same column names and minimum needed information, or the ID/plate/well mapping fails
- Sequence creation is specifically set up for the Orbitrap ID-X (Xcalibur); other instruments need adapted layouts
- Some external database sources need local files or special access and must be disabled in jobs.py if unavailable, which can leave metadata incomplete before ID assignment

## Evidence

- [methods] Sequence creation is set up for the Orbitrap ID-X: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] Metadata template requirement: "Please use the [template](https://docs.google.com/spreadsheets/d/1v6_IlGS3VgycGc-mSSdNeocY-CFXpONVZbuh3XNLX2E/edit?usp=sharing) for your metadata for having same column names and minimum needed"
- [methods] Metadata cleanup flow: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run
  and wait for jobs to be submitted (jobs.py)"
- [methods] Well plate structure: "# Well visualization (piechart if a compound mixture was used in each well)

The example shows a 384 well plate and each well contains 10 different compounds."
