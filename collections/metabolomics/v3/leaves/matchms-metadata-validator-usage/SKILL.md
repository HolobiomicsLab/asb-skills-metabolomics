---
name: matchms-metadata-validator-usage
description: Use when msp mass spectral library records have been annotated with fetched
  or computed metadata (InChI, SMILES, CAS, InChIKey) and each record's attributes
  must be checked for well-formedness before export or downstream use.
when_to_use_negative:
- When the .msp records have not yet been annotated — validation checks the form of
  fetched metadata and presupposes the iterative annotation steps already ran.
- When the goal is to fetch or convert identifiers (e.g. compound_name_to_inchi via
  CIR/CTS/IDSM) rather than to verify the form of already-present attributes.
- 'When validation must be tracked in the log: per article discussion, attribute validation
  tracking was still an open issue (#68).'
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: matchms
  role: Provides the data IO for .msp files and the validators used to check the correct
    form of metadata attributes such as InChI, InChIKey, SMILES, and CAS numbers.
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: MSMetaEnhancer
  role: Produces the annotated .msp records whose metadata are subsequently validated;
    it embeds this validation stage after its asynchronous, iterative annotation steps.
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: Python
  role: Implementation language in which the per-record validation loop over matchms
    validators is written and the validation report is serialized.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:a37a032df204a70cb9d8146917ad8b3416b91e90c1c280775599b78e79fad240
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - Python
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# matchms-metadata-validator-usage

## Summary

Validates that metadata attributes attached to mass spectral library records (InChI, InChIKey, SMILES, CAS numbers) are in the correct form by applying matchms validators to each spectrum record after MSMetaEnhancer's iterative annotation steps.

## When to use

Use when .msp mass spectral library records have been annotated with fetched or computed metadata (InChI, SMILES, CAS, InChIKey) and each record's attributes must be checked for well-formedness before export or downstream use.

## When NOT to use

- When the .msp records have not yet been annotated — validation checks the form of fetched metadata and presupposes the iterative annotation steps already ran.
- When the goal is to fetch or convert identifiers (e.g. compound_name_to_inchi via CIR/CTS/IDSM) rather than to verify the form of already-present attributes.
- When validation must be tracked in the log: per article discussion, attribute validation tracking was still an open issue (#68).

## Inputs

- Annotated .msp mass spectral library file (output of MSMetaEnhancer annotate_spectra)
- Per-record metadata attributes: InChI, InChIKey, SMILES, CAS number

## Outputs

- Per-record validation report (e.g. JSON) mapping each spectrum record to which attributes passed/failed the correct-form check
- Optional detailed log file entries recording annotation/validation issues

## How to apply

Run this step after MSMetaEnhancer's annotation phase, in which the tool iteratively executes all converter jobs (e.g. ('inchi', 'canonical_smiles', 'IDSM')) per entry until no new metadata is found. Load the annotated .msp file with matchms so each spectrum record exposes its metadata fields. For each record, apply the matchms validators for InChI, InChIKey, SMILES, and CAS number to the corresponding metadata attributes, since values fetched from services like CIR, CTS, PubChem, IDSM, and BridgeDb are not guaranteed to be well-formed. Collect per-record pass/fail outcomes for each attribute; optionally feed failures into the detailed log file MSMetaEnhancer can emit. Serialize the per-record validation report (e.g. JSON mapping spectrum record to validation results) so downstream QC can identify records whose fetched metadata must be corrected or dropped.

## Related tools

- **matchms** (Provides the data IO for .msp files and the validators used to check the correct form of metadata attributes such as InChI, InChIKey, SMILES, and CAS numbers.) — https://github.com/RECETOX/MSMetaEnhancer
- **MSMetaEnhancer** (Produces the annotated .msp records whose metadata are subsequently validated; it embeds this validation stage after its asynchronous, iterative annotation steps.) — https://github.com/RECETOX/MSMetaEnhancer
- **Python** (Implementation language in which the per-record validation loop over matchms validators is written and the validation report is serialized.)

## Examples

```
app.load_data('annotated_spectra.msp', file_format='msp'); app.curate_metadata(); # then apply matchms validators (InChI, InChIKey, SMILES, CAS) per record and save the validation report as JSON
```

## Evaluation signals

- Every spectrum record in the .msp file appears in the validation report — no record is skipped.
- Each of the four attributes (InChI, InChIKey, SMILES, CAS number) has an explicit pass/fail outcome per record, not a missing entry.
- Values flagged as failing should be malformed (e.g. CAS numbers not in the curatable form that app.curate_metadata() targets); spot-check a sample by parsing the identifier format.
- Records whose metadata passed validation should be unchanged relative to the annotated .msp input.
- Validation issues surface in the optional detailed log file, consistent with MSMetaEnhancer's issue-recording behavior.

## Limitations

- Attribute validation tracking in the log was not yet implemented (open issue #68).
- Validation only checks the correct form of metadata, not its chemical correctness against the spectrum.
- Because metadata come from external services (CIR, CTS, PubChem, IDSM, BridgeDb), some service behaviors are incompletely specified — e.g. CIR taking only the first result when there are multiple hits (#69), and PubChem's generic SMILES vs ISOMERIC/CANONICAL SMILES handling (#67) — which can propagate malformed or ambiguous values into the validation stage.

## Evidence

- [abstract] validation stage follows iterative annotation: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] iterative per-record annotation precedes validation: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."
- [readme] curate/fix CAS numbers before validation: [evidence span withheld — anchor status: external_doc]
- [discussion] validation tracking in log not yet implemented: "tracking of attributes validation in log [#68](https://github.com/RECETOX/MSMetaEnhancer/issues/68)"
