---
name: spectral-library-metadata-annotation
description: Use when an .msp mass spectral library file contains records lacking
  metadata such as SMILES, InChI, InChIKey, CAS numbers, chemical formula, or molecular
  weight, and the records need post-processing to enrich them.
when_to_use_negative:
- Records already carry complete, validated metadata — re-annotation will only re-query
  or duplicate existing fields.
- The target attribute cannot be derived from any available source attribute or supported
  service (e.g. no SMILES present for RDKit-based molecular weight computation).
- The input is not a mass spectral library file in a supported format (MSMetaEnhancer's
  IO is matchms-based .msp).
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MSMetaEnhancer
  role: 'Core annotation engine: loads/saves .msp, schedules asynchronous annotation
    jobs, caches results, validates metadata, and hosts web and compute converters'
  repo: github.com/RECETOX/MSMetaEnhancer
- name: matchms
  role: Provides data IO for .msp format and validators used to check the correct
    form of obtained metadata
- name: RDKit
  role: Local computation of molecular weight from SMILES via MolFromSmiles inside
    a ComputeConverter, avoiding external database queries
- name: CIR
  role: External Chemical Identifier Resolver service for converting between chemical
    identifiers (e.g. compound_name_to_inchi)
  repo: mcs07/CIRpy
- name: Galaxy
  role: Wrapper providing a user-friendly interface and reproducible data processing
    of MSMetaEnhancer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:222728b1befeffa564d1c7574cca2a1d9bd8af886c5a540f184a4fb0aef4734e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIR
  - Galaxy
  - MSMetaEnhancer
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-library-metadata-annotation

## Summary

Annotate mass spectral library records in .msp format by adding missing metadata (SMILES, InChI, CAS numbers, molecular weight) using MSMetaEnhancer, which fetches identifiers from external services (CIR, CTS, PubChem, IDSM, BridgeDb) or computes them locally (e.g., molecular weight from SMILES via RDKit).

## When to use

Use when an .msp mass spectral library file contains records lacking metadata such as SMILES, InChI, InChIKey, CAS numbers, chemical formula, or molecular weight, and the records need post-processing to enrich them.

## When NOT to use

- Records already carry complete, validated metadata — re-annotation will only re-query or duplicate existing fields.
- The target attribute cannot be derived from any available source attribute or supported service (e.g. no SMILES present for RDKit-based molecular weight computation).
- The input is not a mass spectral library file in a supported format (MSMetaEnhancer's IO is matchms-based .msp).

## Inputs

- .msp mass spectral library file (matchms-format spectra records, annotated upstream with SMILES or other identifiers)
- list of services to use (e.g. ['CTS', 'CIR', 'IDSM', 'PubChem', 'BridgeDb', 'RDKit'])
- list of annotation jobs as (source attribute, target attribute, service) tuples

## Outputs

- annotated .msp file with added metadata such as SMILES, InChI, InChIKey, CAS number, formula, iupac_name, molecular_weight
- optional detailed log file recording annotation issues

## How to apply

Load the .msp file with MSMetaEnhancer's load_data (matchms-based IO, file_format='msp'), optionally curate metadata (e.g., fix CAS numbers), then register converters (CTS, CIR, IDSM, PubChem, BridgeDb, RDKit) via ConverterBuilder.register and specify annotation jobs as (source attribute, target attribute, service) tuples. Run the asynchronous annotation with asyncio.run(app.annotate_spectra(services, jobs)); MSMetaEnhancer iteratively executes all steps per entry until no new metadata is found, uses caching of results, and throttles rate-limited services. For metadata that should not come from external queries — e.g., molecular weight derived from existing SMILES — write a ComputeConverter subclass whose conversions list maps ('smiles', 'molecular_weight', 'conversion_method') and parses SMILES locally with RDKit's MolFromSmiles, returning {'molecular_weight': value} or an empty dict when SMILES is missing. Obtained metadata are validated with matchms validators, annotation issues can be recorded in an optional detailed log file, and results are exported with save_data to a new .msp file.

## Related tools

- **MSMetaEnhancer** (Core annotation engine: loads/saves .msp, schedules asynchronous annotation jobs, caches results, validates metadata, and hosts web and compute converters) — github.com/RECETOX/MSMetaEnhancer
- **matchms** (Provides data IO for .msp format and validators used to check the correct form of obtained metadata)
- **RDKit** (Local computation of molecular weight from SMILES via MolFromSmiles inside a ComputeConverter, avoiding external database queries)
- **CIR** (External Chemical Identifier Resolver service for converting between chemical identifiers (e.g. compound_name_to_inchi)) — mcs07/CIRpy
- **Galaxy** (Wrapper providing a user-friendly interface and reproducible data processing of MSMetaEnhancer)

## Examples

```
app = Application(); app.load_data('sample.msp', file_format='msp'); services = ['CTS', 'CIR', 'IDSM', 'PubChem', 'BridgeDb', 'RDKit']; jobs = [('inchi', 'canonical_smiles', 'IDSM')]; asyncio.run(app.annotate_spectra(services, jobs)); app.save_data('sample_out.msp', file_format='msp')
```

## Evaluation signals

- Output .msp file exists, parses as matchms .msp, and contains the added metadata fields (e.g. molecular_weight) on the annotated records.
- Locally computed values match independent recomputation — e.g. molecular weights from a sample of SMILES strings reproduce RDKit's values.
- Obtained metadata pass matchms validators ensuring their correct form.
- Iterative annotation terminates: repeated execution of all steps yields no new metadata (fixed-point of the annotation process).
- Annotation issues (missing responses, invalid values) are captured in the optional detailed log file.

## Limitations

- External services can return multiple hits — e.g. CIR conversions should take only the first result when there are multiple hits (open issue #69).
- PubChem SMILES handling is incomplete: ISOMERIC_SMILES and CANONICAL_SMILES are not yet distinguished from generic SMILES (open issue #67).
- Tracking of attribute validation in the log is not yet implemented (open issue #68).
- Metadata such as chemical formula, SMILES, InChI, and spectrum origin are not always present in mass spectral library records, so upstream post-processing is required.

## Evidence

- [abstract] It annotates given mass spectra records in the library file by adding missing metadata such as SMILES, In ChI, and CAS numbers to the individual entries.: "It annotates given mass spectra records in the library file by adding missing metadata such as SMILES, In ChI, and CAS numbers to the individual entries."
- [abstract] instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES): "instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES)"
- [abstract] The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found.: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."
- [abstract] app.load_spectra('input_spectra_file.msp', file_format='msp'): "app.load_spectra('input_spectra_file.msp', file_format='msp')"
- [abstract] the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task): "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [readme] ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit]): [evidence span withheld — anchor status: external_doc]
- [readme] asyncio.run(app.annotate_spectra(services, jobs)): "asyncio.run(app.annotate_spectra(services, jobs))"
