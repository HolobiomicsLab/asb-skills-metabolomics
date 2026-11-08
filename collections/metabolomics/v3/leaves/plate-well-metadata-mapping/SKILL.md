---
name: plate-well-metadata-mapping
description: Use when converting a cleaned sample metadata sheet (384-well or other
  plate layout, flow injection MSn) into an instrument-ready acquisition sequence
  with unique sample IDs, well positions, and per-polarity acquisition file names.
when_to_use_negative:
- Data are raw Thermo RAW files needing conversion to mzML/MGF — use ThermoRawFileParser
  instead of sequence generation
- Metadata still needs database enrichment (LOTUS, DrugBank, DrugCentral) — run the
  metadata cleanup flow first
- The instrument is not an Orbitrap ID-X with Xcalibur; the sequence layout is explicitly
  'setup for the Orbitrap ID-X (Xcalibur)'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Xcalibur
  role: Target sequence-table format; generated sequence.csv is imported into Xcalibur
    for Orbitrap ID-X acquisition
- name: Orbitrap ID-X
  role: Instrument whose flow injection MSn acquisition the sequence is created for
- name: Prefect
  role: Orchestrates the metadata cleanup / sequence creation flows (prefect deploy,
    work pool 'local-work' with concurrency limit 5)
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: pip
  role: Installs the Python requirements for running the pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:c3a0a96d029ab687abbf90bd68cfd6fae352cd1e63bfd811c90f429874a22151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Orbitrap ID-X
  - Prefect
  - Xcalibur
  - pip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# plate-well-metadata-mapping

## Summary

Maps cleaned plate/well metadata (unique sample id, plate_id, well_location) onto an Xcalibur-format acquisition sequence for Orbitrap ID-X flow injection MSn, generating per-well acquisition file names and positive/negative ordering per plate.

## When to use

Use when converting a cleaned sample metadata sheet (384-well or other plate layout, flow injection MSn) into an instrument-ready acquisition sequence with unique sample IDs, well positions, and per-polarity acquisition file names.

## When NOT to use

- Data are raw Thermo RAW files needing conversion to mzML/MGF — use ThermoRawFileParser instead of sequence generation
- Metadata still needs database enrichment (LOTUS, DrugBank, DrugCentral) — run the metadata cleanup flow first
- The instrument is not an Orbitrap ID-X with Xcalibur; the sequence layout is explicitly 'setup for the Orbitrap ID-X (Xcalibur)'

## Inputs

- Cleaned metadata sheet (template-based, e.g. Google Sheets/CSV) with columns: unique sample id, plate_id, well_location
- MS method file name(s) for positive and negative ionization modes
- Acquisition date for file name generation

## Outputs

- sequence.csv — Xcalibur-format sequence table, one row per well, ordered positive then negative mode per plate
- Auto-generated acquisition file names per well following the date-sampleID-method-polarity convention

## How to apply

Load the cleaned metadata sheet with pandas, requiring the columns 'unique sample id', 'plate_id' (the batch identifier / plate name), and 'well_location' (vial or well position), as specified for the Orbitrap ID-X Xcalibur setup. Validate and assign unique sample IDs per well/plate combination so no collisions occur across the batch; if structure information is missing, metadata can be enriched by PubChem Name search beforehand. Auto-generate acquisition file names composed of date, sample ID, method, and polarity for each well. Sort wells so that within each plate_id all positive-ionization-mode entries precede negative-ionization-mode entries, then write the ordered rows into an Xcalibur-format sequence table compatible with Orbitrap ID-X import and save it as sequence.csv. The flow can be served and scheduled locally via Prefect (prefect server, work pool with concurrency limit 5), consistent with the metadata cleanup pipeline in jobs.py.

## Related tools

- **Xcalibur** (Target sequence-table format; generated sequence.csv is imported into Xcalibur for Orbitrap ID-X acquisition)
- **Orbitrap ID-X** (Instrument whose flow injection MSn acquisition the sequence is created for)
- **Prefect** (Orchestrates the metadata cleanup / sequence creation flows (prefect deploy, work pool 'local-work' with concurrency limit 5)) — https://github.com/corinnabrungs/msn_tree_library
- **pip** (Installs the Python requirements for running the pipeline)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- sequence.csv contains exactly one row per well with all required Xcalibur columns populated
- All sample IDs are unique per well/plate combination with no collisions across the batch
- Every acquisition file name follows the date-sampleID-method-polarity convention
- Within each plate_id, rows are ordered positive-ionization mode first, then negative-ionization mode
- Required columns 'unique sample id', 'plate_id', and 'well_location' are present and non-empty in the input metadata

## Limitations

- Sequence layout is specific to the Orbitrap ID-X (Xcalibur); other instruments or vendor sequence formats are not covered
- External database enrichment (Dictionary of Natural Products, DrugBank) requires local files or special access, otherwise these must be set to False in jobs.py
- Metadata must follow the shared template with the same column names and minimum needed information; non-conforming sheets will fail the required-column checks

## Evidence

- [methods] The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] Please use the [template]...for your metadata for having same column names and minimum needed information for the query.: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [readme] If no structure information is provided, it is queried from PubChem by Name search.: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] some need a local file and/or special access otherwise set it to False in the jobs.py: "some need a local file and/or special access otherwise set it to False in the jobs.py"
