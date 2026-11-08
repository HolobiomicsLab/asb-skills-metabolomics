---
name: chemical-identifier-format-validation
description: Use when a mass spectral library file has been annotated with externally
  fetched metadata (e.g. via MSMetaEnhancer services CIR, CTS, IDSM, PubChem, BridgeDb)
  and you must check each InChI, InChIKey, SMILES, or CAS attribute for well-formedness
  before downstream use.
when_to_use_negative:
- The input is a raw or unannotated .msp file without chemical identifier metadata
  to check — run MSMetaEnhancer annotation first.
- You need to verify that an identifier corresponds to the correct compound (identity/ground-truth
  checking) — validators only ensure the correct form of attributes.
- The data are raw spectra in mzML or similar acquisition formats rather than an annotated
  spectral library file.
tools:
- name: matchms
  role: Provides the validators used to check the correct form of InChI, InChIKey,
    SMILES, and CAS metadata; also used for .msp data IO.
- name: MSMetaEnhancer
  role: Produces the annotated records whose metadata are validated, and includes
    curate_metadata() for pre-validation fixing of attributes such as CAS numbers.
  repo: github.com/RECETOX/MSMetaEnhancer
- name: Python
  role: Execution language for the validation loop and JSON report generation.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:faba0193966c7cb1028a7d745335c9525e453351a94e68532fd7f45eb8f5c2cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSMetaEnhancer
  - Python
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chemical-identifier-format-validation

## Summary

Validates that chemical identifier metadata (InChI, InChIKey, SMILES, CAS number) fetched during .msp annotation are in their correct syntactic form, using the validators provided by the matchms package.

## When to use

Use when a mass spectral library file has been annotated with externally fetched metadata (e.g. via MSMetaEnhancer services CIR, CTS, IDSM, PubChem, BridgeDb) and you must check each InChI, InChIKey, SMILES, or CAS attribute for well-formedness before downstream use.

## When NOT to use

- The input is a raw or unannotated .msp file without chemical identifier metadata to check — run MSMetaEnhancer annotation first.
- You need to verify that an identifier corresponds to the correct compound (identity/ground-truth checking) — validators only ensure the correct form of attributes.
- The data are raw spectra in mzML or similar acquisition formats rather than an annotated spectral library file.

## Inputs

- annotated .msp mass spectral library file (matchms-readable, produced by MSMetaEnhancer after annotation)
- per-record metadata attributes: InChI, InChIKey, canonical/isomeric SMILES, CAS number

## Outputs

- JSON validation report mapping each spectrum record to the pass/fail status of its InChI, InChIKey, SMILES, and CAS attributes
- optional detailed log file recording annotation and validation issues

## How to apply

Run this step after the iterative annotation stage, where MSMetaEnhancer executes all conversion jobs per entry until no new metadata is found. Load the annotated .msp file with matchms (which the article states is used for data IO of the open, text-based .msp format) to obtain the metadata fields of each spectrum record. For each record, apply the matchms validators for InChI, InChIKey, SMILES, and CAS number to the corresponding metadata attributes — note the validators check 'correct form', not whether the identifier refers to the right compound. Collect the pass/fail outcome per attribute per record, and save a per-record validation report as a JSON file mapping each spectrum record to its validation results. Because fetched values can arrive malformed from rate-limited web services (CIR, CTS, IDSM, PubChem, BridgeDb), this per-record check is the last gate before exporting the annotated .msp file.

## Related tools

- **matchms** (Provides the validators used to check the correct form of InChI, InChIKey, SMILES, and CAS metadata; also used for .msp data IO.)
- **MSMetaEnhancer** (Produces the annotated records whose metadata are validated, and includes curate_metadata() for pre-validation fixing of attributes such as CAS numbers.) — github.com/RECETOX/MSMetaEnhancer
- **Python** (Execution language for the validation loop and JSON report generation.)

## Examples

```
from matchms.importing import load_from_msp; spectra = list(load_from_msp('annotated_spectra_file.msp'))
```

## Evaluation signals

- Every spectrum record in the annotated .msp file appears exactly once as a key in the JSON validation report.
- Each record's report covers all four attribute types (InChI, InChIKey, SMILES, CAS number) with an explicit pass or fail outcome, rather than silently omitting missing attributes.
- No exported .msp record contains an attribute that failed the correct-form check without either being corrected (e.g. via curate_metadata for CAS) or flagged in the log.
- Validation failures correlate with annotation issues recorded in the detailed log file, so each failed attribute can be traced to the service that supplied it (CIR, CTS, IDSM, PubChem, BridgeDb).

## Limitations

- Validators check only the correct form of identifiers, not whether the identifier describes the intended compound.
- Tracking of attribute validation in the log file was not yet implemented at time of writing (issue #68).
- Coverage is limited to the attribute types with matchms validators; other fetched metadata (e.g. iupac_name, formula) are not format-checked by this step.
- Upstream service quirks (e.g. CIR returning multiple hits, PubChem generic SMILES vs ISOMERIC/CANONICAL_SMILES, issues #69 and #67) can inject inconsistent metadata that validators then flag.

## Evidence

- [abstract] matchms validators used for correct-form checking of fetched metadata: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [readme] metadata types subject to validation: "It adds metadata like SMILES, InChI, and CAS number fetched from the following services"
- [abstract] per-record iterative annotation precedes validation: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."
- [abstract] issue logging of validation problems: [evidence span withheld — anchor status: not_found]
- [discussion] validation tracking gap: "tracking of attributes validation in log [#68]"
