---
name: matchms-spectra-format-validation
description: Use when a .msp spectral library's metadata (e.g. InChI, SMILES, CAS
  number, formula) must be checked or curated for correct form, before or after enrichment
  via external identifier-conversion services.
when_to_use_negative:
- The input is not a text-based .msp spectral library (e.g. raw instrument data or
  mzML files), since this skill targets metadata validation of library records
- The task is identifier conversion or enrichment itself rather than checking metadata
  correctness — use the web converters (CIR, CTS, IDSM, PubChem, BridgeDb) instead
- Metadata is already known to be valid and no curation or annotation step is required
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3361
tools:
- name: matchms
  role: Provides the data IO for .msp files and the validators used to ensure metadata
    fields are in their correct form
- name: MSMetaEnhancer
  role: Application wrapper that triggers matchms validators during curate_metadata()
    and annotate_spectra() on .msp records
  repo: https://github.com/RECETOX/MSMetaEnhancer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:890aa432da1f5706d650be5de838ba8ba9173c47ea1920ea7e0fa0c291c5c53b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - CIR
  - CTS
  - IDSM
  - MSMetaEnhancer
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# matchms-spectra-format-validation

## Summary

Validates that metadata fields (InChI, InChIKey, canonical SMILES, IUPAC name, formula, CAS numbers) in .msp mass spectral library records are in their correct form, using the validators shipped with matchms as invoked by MSMetaEnhancer during annotation and curation.

## When to use

Use when a .msp spectral library's metadata (e.g. InChI, SMILES, CAS number, formula) must be checked or curated for correct form, before or after enrichment via external identifier-conversion services.

## When NOT to use

- The input is not a text-based .msp spectral library (e.g. raw instrument data or mzML files), since this skill targets metadata validation of library records
- The task is identifier conversion or enrichment itself rather than checking metadata correctness — use the web converters (CIR, CTS, IDSM, PubChem, BridgeDb) instead
- Metadata is already known to be valid and no curation or annotation step is required

## Inputs

- .msp mass spectral library file (text-based, read via matchms IO)
- Spectral records with metadata fields such as compound_name, inchikey, inchi, CAS number
- Annotation jobs as (source attribute, target attribute, service) tuples, e.g. ('inchikey', 'inchi', 'CIR')

## Outputs

- Validated and curated .msp file with metadata fields (inchi, iupac_name, canonical_smiles, formula) in correct form
- Optional detailed log file recording annotation and validation issues per record

## How to apply

Load the .msp spectral library through MSMetaEnhancer's Application, which relies on matchms for data IO of the open, text-based .msp format. First run app.curate_metadata() to fix malformed existing metadata such as incorrectly formatted CAS numbers. After running asyncio.run(app.annotate_spectra(services, jobs)) with services like CIR, CTS, and IDSM, every newly derived metadata value is automatically passed through matchms validators before the record is accepted. The tool processes the library by iteratively executing all annotation steps per entry until no new metadata is found, so validation applies at each round to fields such as inchi, iupac_name, canonical_smiles, and formula. Any annotation or validation issues can be captured in an optional detailed log file, and the validated records are exported with app.save_data/save_spectra to an output .msp file.

## Related tools

- **matchms** (Provides the data IO for .msp files and the validators used to ensure metadata fields are in their correct form)
- **MSMetaEnhancer** (Application wrapper that triggers matchms validators during curate_metadata() and annotate_spectra() on .msp records) — https://github.com/RECETOX/MSMetaEnhancer

## Examples

```
app.load_data('sample.msp', file_format='msp'); app.curate_metadata(); asyncio.run(app.annotate_spectra(services, jobs)); app.save_data('sample_out.msp', file_format='msp')
```

## Evaluation signals

- Every output .msp record retains its input fields and carries newly derived inchi, iupac_name, canonical_smiles, and formula values that pass matchms validators
- Record count in the annotated output .msp equals the input record count (no entries dropped during validation/iteration)
- Curated CAS numbers are in a normalized form after app.curate_metadata()
- Records failing validation are traceable via the optional detailed log file rather than silently missing fields
- Iterative annotation terminates (no new metadata found) rather than oscillating on invalid values

## Limitations

- Tracking of attribute validation in the log file was not yet implemented (issue #68)
- PubChem SMILES handling was incomplete — generic SMILES instead of distinguishing ISOMERIC_SMILES and CANONICAL_SMILES (issue #67)
- CIR behavior for multiple hits was underspecified — only the first result is taken in CIR conversions (issue #69), so a validated value may not be the intended one

## Evidence

- [abstract] validators employed before saving annotated metadata: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] matchms used for .msp data IO: "The package uses matchms (Huber et al., 2020) for data IO and supports the open, text-based .mspformat"
- [discussion] validation logging not yet implemented: "tracking of attributes validation in log [#68](https://github.com/RECETOX/MSMetaEnhancer/issues/68)"
- [readme] README curate_metadata example: "# curate given metadata (e.g. fix CAS numbers)
app.curate_metadata()"
