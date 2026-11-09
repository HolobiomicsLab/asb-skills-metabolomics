---
name: asynchronous-web-service-querying
description: Use when a .msp mass spectral library has records missing metadata such
  as SMILES, InChI, InChIKey, IUPAC name, CAS number, or formula, and those fields
  can be derived from existing fields via external chemical identifier services.
when_to_use_negative:
- When the required metadata is already present in the .msp records and no identifier
  conversion is needed — the services are only for filling missing fields
- When the target identifier cannot be computed locally or obtained from a supported
  service (MSMetaEnhancer supports CIR, CTS, PubChem, IDSM, BridgeDb, and computed
  conversions like molecular weight from SMILES via RDKit), not arbitrary web APIs
- When the user needs a no-code workflow and reproducibility through a web interface
  — use the Galaxy wrapper instead of writing asyncio scripts directly
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_3316
tools:
- name: MSMetaEnhancer
  role: Application object orchestrating the asynchronous annotation of .msp spectra
    records; load_data, annotate_spectra, and save_data drive the workflow
  repo: github.com/RECETOX/MSMetaEnhancer
- name: CIR
  role: Chemical Identifier Resolver web service queried for identifier conversions
    such as inchikey to inchi (accessed via the CIRpy Python interface)
  repo: https://github.com/mcs07/CIRpy
- name: CTS
  role: Chemical Translation Service queried for jobs such as inchikey to iupac_name
- name: IDSM
  role: Integrated Database for Small Molecules service queried for inchi to canonical_smiles
    and inchi to formula conversions
- name: matchms
  role: Handles .msp data IO and provides validators used to ensure the obtained metadata
    are in their correct form
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:a1836fde52ea505bd935c8458d780420970ba8f1b89fa946e9ad388de2c85c9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIR
  - CTS
  - IDSM
  - MSMetaEnhancer
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# asynchronous-web-service-querying

## Summary

Annotates mass spectral library records in .msp files by asynchronously querying external chemical web services (CIR, CTS, IDSM, PubChem, BridgeDb) to convert one identifier into others (e.g. inchikey to inchi, inchi to canonical_smiles), caching per-compound results and throttling rate-limited services.

## When to use

Use when a .msp mass spectral library has records missing metadata such as SMILES, InChI, InChIKey, IUPAC name, CAS number, or formula, and those fields can be derived from existing fields via external chemical identifier services.

## When NOT to use

- When the required metadata is already present in the .msp records and no identifier conversion is needed — the services are only for filling missing fields
- When the target identifier cannot be computed locally or obtained from a supported service (MSMetaEnhancer supports CIR, CTS, PubChem, IDSM, BridgeDb, and computed conversions like molecular weight from SMILES via RDKit), not arbitrary web APIs
- When the user needs a no-code workflow and reproducibility through a web interface — use the Galaxy wrapper instead of writing asyncio scripts directly

## Inputs

- .msp mass spectral library file (open, text-based format read/written via matchms)
- List of services to query (e.g. ['CIR', 'CTS', 'IDSM', 'PubChem', 'BridgeDb', 'RDKit'])
- List of jobs as (source attribute, target attribute, service) tuples (e.g. ('inchikey', 'inchi', 'CIR'))

## Outputs

- Enriched .msp file (e.g. annotated_spectra_file.msp) with newly derived inchi, iupac_name, canonical_smiles, formula, CAS number metadata fields
- Optional detailed log file recording any issues regarding the annotation process

## How to apply

Instantiate the MSMetaEnhancer Application and load the library with app.load_data('sample.msp', file_format='msp'), optionally curating existing metadata first (e.g. fixing CAS numbers). Register converters via ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit]), then define jobs as (source_attribute, target_attribute, service) tuples, e.g. ('inchikey', 'inchi', 'CIR'), ('inchikey', 'iupac_name', 'CTS'), ('inchi', 'canonical_smiles', 'IDSM'). Run the annotation with asyncio.run(app.annotate_spectra(services, jobs)): the asynchronous implementation allows optimal fetching speed, results for each compound are cached, and rate-limited services are throttled. The tool processes the library by iteratively executing all steps for each entry until no new metadata is found, validates obtained metadata with matchms validators, and optionally records annotation issues in a detailed log file. Finally export the enriched library with app.save_data('sample_out.msp', file_format='msp').

## Related tools

- **MSMetaEnhancer** (Application object orchestrating the asynchronous annotation of .msp spectra records; load_data, annotate_spectra, and save_data drive the workflow) — github.com/RECETOX/MSMetaEnhancer
- **CIR** (Chemical Identifier Resolver web service queried for identifier conversions such as inchikey to inchi (accessed via the CIRpy Python interface)) — https://github.com/mcs07/CIRpy
- **CTS** (Chemical Translation Service queried for jobs such as inchikey to iupac_name)
- **IDSM** (Integrated Database for Small Molecules service queried for inchi to canonical_smiles and inchi to formula conversions)
- **matchms** (Handles .msp data IO and provides validators used to ensure the obtained metadata are in their correct form)

## Examples

```
import asyncio; asyncio.run(app.annotate_spectra(services, jobs))
```

## Evaluation signals

- The output .msp file must contain all input records, with the newly derived inchi, iupac_name, canonical_smiles, and formula metadata fields populated and validated by matchms validators
- Iterative execution must terminate: repeatedly running all annotation steps per entry should eventually yield no new metadata
- Annotation issues (e.g. failed service queries) should appear in the optional detailed log file rather than be silently dropped
- Cached results should prevent repeated queries for metadata related to the same compound across records

## Limitations

- Rate limits of external web services can constrain throughput; the tool throttles requests for such services, so runtime depends on library size and service responsiveness
- CIR behavior when multiple hits are returned is not fully specified — an open issue proposes taking only the first result in CIR conversions (issue #69)
- PubChem SMILES handling is incomplete: support for ISOMERIC_SMILES and CANONICAL_SMILES instead of generic SMILES was still open (issue #67)
- Tracking of attribute validation in the log was not yet implemented (issue #68)
- Metadata such as formula, SMILES, InChI, and spectrum origin are not always present in mass spectral library records, so a failed service query can leave fields empty

## Evidence

- [abstract] asynchronous efficiency: "this task is suitable for the asynchronous approach, making the tool computationally efficient. Additionally, results containing all metadata related to a compound are cached"
- [abstract] validation of metadata: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] iterative processing: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."
- [intro] async motivation: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
- [readme] README usage snippet: "ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit])

app = Application()"
- [readme] README annotation call: "asyncio.run(app.annotate_spectra(services, jobs))"
