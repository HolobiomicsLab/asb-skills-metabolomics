---
name: mass-spectral-library-msp-io
description: Use when input is a mass spectral library file in .msp format whose records
  need loading into a Python tool, enrichment with metadata (SMILES, InChI, CAS),
  or export back to .msp after annotation.
when_to_use_negative:
- Input spectra are in a format other than .msp (e.g. raw vendor formats or mzML);
  MSMetaEnhancer's demonstrated IO supports the .msp format via matchms
- The goal is spectral processing (peak picking, similarity scoring) rather than metadata
  annotation and round-tripping of library records
- Records already contain complete, validated metadata (InChI, SMILES, formula) so
  no external enrichment round-trip is needed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: MSMetaEnhancer
  role: Provides the Application object whose load_data/load_spectra and save_data/save_spectra
    methods perform the .msp round-trip around annotation
  repo: github.com/RECETOX/MSMetaEnhancer
- name: matchms
  role: Underlies the data IO for the open, text-based .msp format and supplies validators
    used to check metadata correctness
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:3b44aa5404da7594403593549bff342e7fb92fc53c1a3cb17631a813c3e4ee0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIR
  - CTS
  - IDSM
  - MSMetaEnhancer
  - matchms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-spectral-library-msp-io

## Summary

Reading and writing open text-based .msp mass spectral library files through matchms-backed IO in MSMetaEnhancer, so records can be loaded for metadata enrichment and the enriched library exported back to .msp.

## When to use

Use when input is a mass spectral library file in .msp format whose records need loading into a Python tool, enrichment with metadata (SMILES, InChI, CAS), or export back to .msp after annotation.

## When NOT to use

- Input spectra are in a format other than .msp (e.g. raw vendor formats or mzML); MSMetaEnhancer's demonstrated IO supports the .msp format via matchms
- The goal is spectral processing (peak picking, similarity scoring) rather than metadata annotation and round-tripping of library records
- Records already contain complete, validated metadata (InChI, SMILES, formula) so no external enrichment round-trip is needed

## Inputs

- Mass spectral library file in open, text-based .msp format (e.g. input_spectra_file.msp)
- Optional file_format specifier ('msp') passed to load_data/load_spectra

## Outputs

- Enriched .msp library file (e.g. annotated_spectra_file.msp) containing all input records with newly derived metadata fields (inchi, iupac_name, canonical_smiles, formula, CAS numbers)
- Optional detailed log file recording annotation issues

## How to apply

Instantiate the MSMetaEnhancer Application object and load the library with app.load_data('sample.msp', file_format='msp') (the paper's example uses app.load_spectra); matchms handles the data IO for the open, text-based .msp format. Once loaded, each spectrum record becomes a dictionary of metadata plus peaks that downstream converters (CIR, CTS, IDSM, PubChem, BridgeDb, RDKit) can annotate. After asynchronous annotation completes, export with app.save_data('sample_out.msp', file_format='msp'), preserving all input records and appending newly derived metadata fields such as inchi, iupac_name, canonical_smiles, and formula. Optionally enable a detailed log file to record any annotation issues encountered between load and save. Validate that obtained metadata are in correct form using matchms validators before or during export.

## Related tools

- **MSMetaEnhancer** (Provides the Application object whose load_data/load_spectra and save_data/save_spectra methods perform the .msp round-trip around annotation) — github.com/RECETOX/MSMetaEnhancer
- **matchms** (Underlies the data IO for the open, text-based .msp format and supplies validators used to check metadata correctness)

## Examples

```
app.load_data('sample.msp', file_format='msp'); asyncio.run(app.annotate_spectra(services, jobs)); app.save_data('sample_out.msp', file_format='msp')
```

## Evaluation signals

- The output .msp file parses successfully with matchms and contains the same number of spectrum records as the input file
- Each annotated record carries the newly derived metadata fields (e.g. inchi, iupac_name, canonical_smiles, formula) populated in correct form as validated by matchms validators
- Pre-existing input metadata (names, peaks) is unchanged between input and output .msp records apart from added fields
- Curated fields such as CAS numbers are in correct form after app.curate_metadata() runs between load and save

## Limitations

- Metadata such as chemical formula, SMILES, InChI, and spectrum origin are not always present in mass spectral library records, so loaded files often require post-processing
- Only the open, text-based .msp format is demonstrated for IO; other formats are not covered by the documented workflow
- Attribute validation tracking in logs is not yet implemented (open issue #68), so invalid metadata forms may not be fully traceable

## Evidence

- [abstract] The package uses matchms (Huber et al., 2020) for data IO and supports the open, text-based .mspformat: "The package uses matchms (Huber et al., 2020) for data IO and supports the open, text-based .mspformat"
- [readme] load spectra: "# import your .msp file
app.load_data('sample.msp', file_format='msp')"
- [readme] export annotated file: "# export .msp file
app.save_data('sample_out.msp', file_format='msp')"
- [abstract] validators: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] missing metadata rationale: "While this metadata is mostly accessible from public chemical databases, they are not always present in mass spectral library records."
