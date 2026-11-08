---
name: chemical-identifier-conversion-annotation
description: Use when a .msp mass spectral library file contains records with incomplete
  metadata (e.g. missing SMILES, InChI, or CAS numbers) and known identifiers that
  can be converted via public chemical web services.
when_to_use_negative:
- The input is not a mass spectral library in .msp format (matchms-based IO supports
  the .msp text format, not general SDF/CSV identifier lists).
- Records already contain complete metadata and no new identifier fields are required,
  so no external annotation is needed.
- No network access is possible and no local conversion applies — the web converters
  (CIR, CTS, IDSM, PubChem, BridgeDb) require live queries; only computed conversions
  like RDKit molecular weight from SMILES work offline.
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: MSMetaEnhancer
  role: Main Application orchestrating loading, asynchronous annotation with caching
    and throttling, validation, and export of .msp records
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: matchms
  role: Data IO for .msp files and validators used to check the correct form of obtained
    metadata
- name: CIR
  role: Chemical Identifier Resolver web service queried for identifier conversions
    such as inchikey→inchi
  repo: https://github.com/mcs07/CIRpy
- name: CTS
  role: Chemical Translation Service used for jobs like inchikey→iupac_name
- name: IDSM
  role: Integrated Database for Small Molecules used for inchi→canonical_smiles and
    inchi→formula conversions
- name: RDKit
  role: Local computed converter, e.g. molecular weight from SMILES, instead of querying
    external databases
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:05c1b3545dacee023aa8693f7f3bbf4c60f383e08d097c7026cf5ceb541ce87a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - CIR
  - CTS
  - IDSM
  - MSMetaEnhancer
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemical-identifier-conversion-annotation

## Summary

Annotates records in .msp mass spectral library files with missing chemical metadata (SMILES, InChI, InChIKey, IUPAC name, CAS number, formula) by querying external identifier-conversion services (CIR, CTS, IDSM, PubChem, BridgeDb) asynchronously and validating results with matchms.

## When to use

Use when a .msp mass spectral library file contains records with incomplete metadata (e.g. missing SMILES, InChI, or CAS numbers) and known identifiers that can be converted via public chemical web services.

## When NOT to use

- The input is not a mass spectral library in .msp format (matchms-based IO supports the .msp text format, not general SDF/CSV identifier lists).
- Records already contain complete metadata and no new identifier fields are required, so no external annotation is needed.
- No network access is possible and no local conversion applies — the web converters (CIR, CTS, IDSM, PubChem, BridgeDb) require live queries; only computed conversions like RDKit molecular weight from SMILES work offline.

## Inputs

- .msp mass spectral library file (open, text-based format handled via matchms)
- Jobs list of (source_attribute, target_attribute, service) tuples, e.g. ('inchikey', 'iupac_name', 'CTS')
- Services list of supported converter names, e.g. ['CTS', 'CIR', 'IDSM', 'PubChem', 'BridgeDb', 'RDKit']

## Outputs

- Enriched .msp file with all input records plus newly derived metadata fields (e.g. inchi, iupac_name, canonical_smiles, formula, CAS number)
- Optional detailed log file recording annotation issues

## How to apply

Instantiate the MSMetaEnhancer Application and load the library with app.load_data('sample.msp', file_format='msp'), optionally calling app.curate_metadata() first to fix e.g. malformed CAS numbers. Register the converters via ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit]), then specify the services list and jobs as (source_attribute, target_attribute, service) tuples, e.g. ('inchikey', 'inchi', 'CIR') or ('inchi', 'canonical_smiles', 'IDSM'). Run the annotation asynchronously with asyncio.run(app.annotate_spectra(services, jobs)) — the asynchronous design allows optimal fetching speed, and per-compound results are cached while rate-limited services are throttled. The tool iteratively executes all jobs for each entry until no new metadata is found, so chained conversions (e.g. name→InChI→SMILES→molecular weight via RDKit) are resolved automatically. Finally export with app.save_data('sample_out.msp', file_format='msp'); derived metadata is validated to ensure correct form using matchms validators before being written.

## Related tools

- **MSMetaEnhancer** (Main Application orchestrating loading, asynchronous annotation with caching and throttling, validation, and export of .msp records) — https://github.com/RECETOX/MSMetaEnhancer
- **matchms** (Data IO for .msp files and validators used to check the correct form of obtained metadata)
- **CIR** (Chemical Identifier Resolver web service queried for identifier conversions such as inchikey→inchi) — https://github.com/mcs07/CIRpy
- **CTS** (Chemical Translation Service used for jobs like inchikey→iupac_name)
- **IDSM** (Integrated Database for Small Molecules used for inchi→canonical_smiles and inchi→formula conversions)
- **RDKit** (Local computed converter, e.g. molecular weight from SMILES, instead of querying external databases)

## Examples

```
from MSMetaEnhancer import Application; app = Application(); app.load_data('sample.msp', file_format='msp'); asyncio.run(app.annotate_spectra(['CTS', 'CIR', 'IDSM'], [('inchikey', 'inchi', 'CIR'), ('inchi', 'canonical_smiles', 'IDSM')])); app.save_data('sample_out.msp', file_format='msp')
```

## Evaluation signals

- The output .msp contains all original input records with the requested target fields (e.g. inchi, iupac_name, canonical_smiles, formula) populated.
- Each newly derived metadata value passes matchms validators, ensuring correct form of the identifier.
- The iterative annotation loop terminates — a final pass adds no new metadata beyond what earlier passes found.
- The optional log file shows which conversions succeeded/failed per record, allowing issue tracing.
- Cached per-compound results mean repeated identifiers do not trigger duplicate service queries (observable in fetch counts/logs).

## Limitations

- CIR behavior for multiple hits is not fully specified — an open issue proposes taking only the first result when there are multiple hits in CIR conversions (issue #69).
- PubChem SMILES handling is incomplete: supporting ISOMERIC_SMILES and CANONICAL_SMILES in PubChem instead of generic SMILES is an open issue (#67).
- Tracking of attribute validation in the log is not yet implemented (issue #68).
- External services are rate-limited, so fetching speed depends on throttling and asynchronous execution.

## Evidence

- [abstract] abstract: "It annotates given mass spectra records in the library file by adding missing metadata such as SMILES, In ChI, and CAS numbers to the individual entries."
- [readme] readme: "It adds metadata like SMILES, InChI, and CAS number fetched from the following services:"
