---
name: mass-spectral-library-file-parsing
description: Use when the input is a mass spectral library file in .msp format and
  downstream work (metadata annotation, identifier conversion, or validation of InChI/SMILES/CAS
  attributes) requires the per-record metadata fields to be loaded and exposed first.
when_to_use_negative:
- The input is raw instrument data in a vendor or open raw format rather than an already-compiled
  text-based .msp spectral library file
- The task is identifier conversion or enrichment itself — parsing is only the IO
  prerequisite; conversion work belongs to the CIR/CTS/IDSM/PubChem/BridgeDb converters
- Only peak/spectrum intensities are needed and no record metadata will be read or
  modified
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Data IO engine that parses the .msp format and exposes per-record spectrum
    metadata
- name: MSMetaEnhancer
  role: Python package wrapping matchms IO; its Application API (load_data/save_data)
    reads and writes .msp spectral library files
  repo: github.com/RECETOX/MSMetaEnhancer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:154447894ecc14921789670147d83121dbcd93b0e9af2664adf74d5aa0a9bb94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSMetaEnhancer
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectral-library-file-parsing

## Summary

Loading and parsing mass spectral library records from the open, text-based .msp format using matchms for data IO, so that per-record metadata fields (SMILES, InChI, CAS numbers, etc.) become available for annotation, curation, and validation.

## When to use

Use when the input is a mass spectral library file in .msp format and downstream work (metadata annotation, identifier conversion, or validation of InChI/SMILES/CAS attributes) requires the per-record metadata fields to be loaded and exposed first.

## When NOT to use

- The input is raw instrument data in a vendor or open raw format rather than an already-compiled text-based .msp spectral library file
- The task is identifier conversion or enrichment itself — parsing is only the IO prerequisite; conversion work belongs to the CIR/CTS/IDSM/PubChem/BridgeDb converters
- Only peak/spectrum intensities are needed and no record metadata will be read or modified

## Inputs

- Mass spectral library file in .msp format (text-based, e.g. input_spectra_file.msp)

## Outputs

- In-memory matchms spectrum records with their metadata fields exposed per record (e.g. name, inchi, inchikey, canonical_smiles, cas_number)
- Annotated .msp file written back via app.save_data(..., file_format='msp')

## How to apply

Use matchms as the data IO layer — MSMetaEnhancer delegates parsing of the open, text-based .msp format to matchms, so records are loaded via the Application API rather than by hand-rolling a parser. Call app.load_data('sample.msp', file_format='msp') (older API: app.load_spectra) to obtain the spectrum records and their metadata fields. After loading, curate given metadata before annotation (e.g. fix malformed CAS numbers) with app.curate_metadata(). Each record then carries its metadata attributes that can be iterated for annotation jobs or checked with matchms validators for InChI, InChIKey, SMILES, and CAS number correct-form, per record. Finally write results back with app.save_data('sample_out.msp', file_format='msp') so the parsed-and-enriched library remains in .msp format.

## Related tools

- **matchms** (Data IO engine that parses the .msp format and exposes per-record spectrum metadata)
- **MSMetaEnhancer** (Python package wrapping matchms IO; its Application API (load_data/save_data) reads and writes .msp spectral library files) — github.com/RECETOX/MSMetaEnhancer

## Examples

```
from MSMetaEnhancer import Application; app = Application(); app.load_data('sample.msp', file_format='msp')
```

## Evaluation signals

- Every record in the source .msp file is loaded and its metadata attributes (e.g. inchi, smiles, cas number) are accessible as matchms metadata fields
- Round-trip check: app.save_data(..., file_format='msp') output is itself a valid .msp file that can be re-loaded by matchms
- Per-record validation outcomes show which of the InChI, InChIKey, SMILES, and CAS attributes passed or failed the correct-form check
- Curated metadata (e.g. corrected CAS numbers after app.curate_metadata()) differs from the raw file only where fixes were expected
- A per-record validation report (e.g. JSON mapping each spectrum record to validation results) is produced without records being skipped

## Limitations

- Metadata such as chemical formula, SMILES, InChI, and spectrum origin are not always present in mass spectral library records, so parsing alone does not guarantee complete metadata and post-processing is needed
- Tracking of attribute validation in the log was not yet implemented (issue #68), so validation outcomes may not be fully traceable in logs
- Parsing exposes metadata but does not itself fetch or correct values; enrichment requires the annotation services, some of whose behaviors are under-specified (e.g. CIR multiple-hit handling, issue #69)

## Evidence

- [abstract] matchms as IO layer for .msp: "The package uses matchms (Huber et al., 2020) for data IO and supports the open, text-based .mspformat"
- [abstract] loading spectra is the documented first workflow step: "app.load_spectra('input_spectra_file.msp', file_format='msp')"
- [abstract] package operates on mass spectral library file records: "MSMetaEnhancer is a Python software package for the metadata enrichment of records in mass spectral library files"
- [readme] README shows the load_data call for .msp parsing: "# import your .msp file
app.load_data('sample.msp', file_format='msp')"
- [readme] README shows round-trip export back to .msp: "# export .msp file
app.save_data('sample_out.msp', file_format='msp')"
