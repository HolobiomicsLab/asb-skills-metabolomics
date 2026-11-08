---
name: per-record-annotation-quality-reporting
description: Use when an annotated .msp spectral library must be checked per record
  that SMILES, InChI, InChIKey, or CAS number metadata are well-formed before export
  or downstream use.
when_to_use_negative:
- The .msp file has not yet been annotated — metadata attributes are missing entirely,
  so there is nothing to validate; run MSMetaEnhancer annotation jobs (e.g. ('name',
  'inchi', 'IDSM')) first.
- The goal is to verify that identifiers are chemically correct against a reference
  database, not that they are syntactically well-formed — validators only check form.
- The input is not the .msp mass spectral library format supported by matchms IO.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: matchms
  role: Provides the data IO for .msp files and the per-attribute validators (InChI,
    InChIKey, SMILES, CAS number) used to check metadata form
- name: MSMetaEnhancer
  role: Upstream tool whose iterative annotation of .msp records produces the metadata
    that this validation stage checks
  repo: github.com/RECETOX/MSMetaEnhancer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:35087f0fb8bfde905115db0f63b2b5fbc0fbf788fa951fad4a0df5a30d77ff8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSMetaEnhancer
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-record-annotation-quality-reporting

## Summary

After MSMetaEnhancer annotates .msp mass spectral records, each record's fetched metadata attributes (InChI, InChIKey, SMILES, CAS number) are checked for correct form using matchms validators, producing a per-record pass/fail quality report.

## When to use

Use when an annotated .msp spectral library must be checked per record that SMILES, InChI, InChIKey, or CAS number metadata are well-formed before export or downstream use.

## When NOT to use

- The .msp file has not yet been annotated — metadata attributes are missing entirely, so there is nothing to validate; run MSMetaEnhancer annotation jobs (e.g. ('name', 'inchi', 'IDSM')) first.
- The goal is to verify that identifiers are chemically correct against a reference database, not that they are syntactically well-formed — validators only check form.
- The input is not the .msp mass spectral library format supported by matchms IO.

## Inputs

- annotated .msp mass spectral library file (MSMetaEnhancer output containing SMILES, InChI, InChIKey, and/or CAS number metadata per record)

## Outputs

- JSON validation report mapping each spectrum record to pass/fail outcomes for InChI, InChIKey, SMILES, and CAS number form checks

## How to apply

Run this stage only after the iterative annotation loop has completed, i.e. after MSMetaEnhancer has processed each entry until no new metadata is found. Load the annotated .msp file with matchms to obtain the metadata fields of each spectrum record, then apply the matchms validators for InChI, InChIKey, SMILES, and CAS number to the corresponding metadata attribute of each record. Collect the validation outcome per attribute per record, distinguishing which attributes passed the correct-form check and which failed. Rationale: the article states that obtained metadata are validated to ensure their correct form and that matchms validators are the employed mechanism, so this step guards against malformed identifiers (e.g. syntactically invalid CAS numbers or SMILES) returned by external services such as CIR, CTS, PubChem, IDSM, or BridgeDb. Serialize the collected outcomes as a JSON file mapping each spectrum record to its validation results so that downstream agents can filter or repair failing records. Note that per-attribute validation tracking inside the MSMetaEnhancer log is not yet implemented (issue #68), so the report must be assembled in a separate step.

## Related tools

- **matchms** (Provides the data IO for .msp files and the per-attribute validators (InChI, InChIKey, SMILES, CAS number) used to check metadata form)
- **MSMetaEnhancer** (Upstream tool whose iterative annotation of .msp records produces the metadata that this validation stage checks) — github.com/RECETOX/MSMetaEnhancer

## Evaluation signals

- The JSON report contains an entry for every spectrum record present in the annotated .msp file (record counts match).
- Each record entry carries an explicit outcome for each of the four checked attributes: InChI, InChIKey, SMILES, and CAS number.
- Known-malformed values (e.g. CAS numbers lacking the correct dash-digit form, empty or placeholder SMILES) are flagged as failures rather than silently passed.
- Spot-checking failing records against matchms validator behavior on the same attribute reproduces the reported pass/fail outcome.
- Records whose metadata failed validation are excluded or repaired before the final .msp export, and the exported file contains only well-formed identifiers.

## Limitations

- Tracking of attribute validation in the MSMetaEnhancer log file is not yet implemented (issue #68), so validation reporting must be done outside the tool's logging.
- Validation only ensures the correct form of metadata attributes; it does not confirm correctness against the source chemical databases (CIR, CTS, PubChem, IDSM, BridgeDb, HMDB).
- Service-specific ambiguities upstream (e.g. CIR returning multiple hits, PubChem generic SMILES vs ISOMERIC/CANONICAL SMILES, issues #69 and #67) can propagate values that pass form validation but are still unsuitable.

## Evidence

- [abstract] matchms validators check metadata form after iterative annotation: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] validation runs per entry after iterative annotation loop: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."
- [discussion] attribute validation tracking in log is an open issue: "tracking of attributes validation in log [#68](https://github.com/RECETOX/MSMetaEnhancer/issues/68)"
- [readme] MSMetaEnhancer annotates .msp records with SMILES, InChI, CAS from external services: "It adds metadata like SMILES, InChI, and CAS number fetched from the following services: [CIR](https://cactus.nci.nih.gov/chemical/structure_documentation), [CTS](https://cts.fiehnlab.ucdavis.edu/),"
