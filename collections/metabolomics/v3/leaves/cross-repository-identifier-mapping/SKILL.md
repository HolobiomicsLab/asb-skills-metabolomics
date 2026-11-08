---
name: cross-repository-identifier-mapping
description: Use when input is a metabolomics or proteomics USI string (e.g. mzspec:MASSBANK::accession:...
  or mzspec:MSV...:file:scan:N) that must be parsed into collection/msRun/indexType/indexNumber
  fields and mapped to the correct repository or spectral library service.
when_to_use_negative:
- Input spectra are local files (mzML, MGF) rather than publicly deposited identifiers
  — parse and visualize them directly instead of routing through repository USIs
- Identifiers are SPLASH hashes rather than USIs — SPLASH generation requires a complex
  hashing operation and follows a different, database-independent scheme
- The task is spectrum rendering/plotting customization (mz_min, mz_max, annotate_peaks,
  mirror plots) rather than identifier decomposition and routing
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose USI extension defines the metabolomics USI formatting rules
    and dereferences parsed identifiers via /spectrum/, /png/, /svg/, /json/, /csv/,
    /qrcode/, /proxi/v0.1/spectra, and /mirror/ endpoints
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: HUPO-PSI identifier specification extended for metabolomics; supplies the
    mzspec field structure being parsed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:b33eec456f061b1056690d805fa8ecaa4e7bdf9ec5e220da5060b79f8f44f6d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - USI
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-repository-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Decomposes metabolomics Universal Spectrum Identifiers (USIs) of the form mzspec:<collection>:<msRun>:<indexType>:<indexNumber> into their four colon-delimited fields and routes each identifier to its backing data service (MassBank, MS2LDA, GNPS, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange), as implemented in the Metabolomics Spectrum Resolver's USI extension.

## When to use

Use when input is a metabolomics or proteomics USI string (e.g. mzspec:MASSBANK::accession:... or mzspec:MSV...:file:scan:N) that must be parsed into collection/msRun/indexType/indexNumber fields and mapped to the correct repository or spectral library service.

## When NOT to use

- Input spectra are local files (mzML, MGF) rather than publicly deposited identifiers — parse and visualize them directly instead of routing through repository USIs
- Identifiers are SPLASH hashes rather than USIs — SPLASH generation requires a complex hashing operation and follows a different, database-independent scheme
- The task is spectrum rendering/plotting customization (mz_min, mz_max, annotate_peaks, mirror plots) rather than identifier decomposition and routing

## Inputs

- USI strings in mzspec:<collection>:<msRun>:<indexType>:<indexNumber> format (e.g. MassBank, MOTIFDB, GNPS task, GNPS library, PXD, MSV identifiers)
- USI formatting rules for MassBank library spectra, GNPS analysis task spectra, MS2LDA MOTIFDB entries, GNPS library spectra, and MetaboLights / Metabolomics Workbench / MassIVE repository schemas

## Outputs

- Parsed USI components (collection, msRun, indexType, indexNumber) per identifier
- Resolved backing resource per identifier (data repositories: MassIVE, MetaboLights, Metabolomics Workbench; reference spectral libraries: MassBank, MoNA; online informatics pipelines)
- Structured table mapping each USI to its collection type and backing resource (e.g. usi_collection_map.csv)

## How to apply

First split each USI on colons into collection, msRun, indexType, and indexNumber, handling the special cases where the msRun field is empty (MassBank accessions like mzspec:MASSBANK::accession:SM858102 and MOTIFDB accessions like mzspec:MOTIFDB::accession:171163) or where msRun is a library name that is ignored (GNPS library spectra, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077). Next classify the collection by the documented formatting patterns: 'TASK-<task ID>-<filename>' marks GNPS/MassIVE analysis task spectra (with '/specs_ms.mgf' indicating a file-level rather than collection-level target), 'MSV...' collections map to MassIVE/GNPS, MetaboLights, or Metabolomics Workbench repository data depending on the dataset, and 'PXD...' accessions map to ProteomeXchange. Then resolve each collection to its backing resource consistent with the Metabolomics Spectrum Resolver's USI extension (MASSBANK -> MassBank, MOTIFDB -> MS2LDA, GNPS -> GNPS task/library, MSV -> MassIVE/MetaboLights/Metabolomics Workbench). Finally emit the parsed components and resolved resource for every input USI so identifiers can be dereferenced via the resolver's /spectrum/, /png/, /svg/, /json/, /csv/, /proxi/v0.1/spectra, or /mirror/ endpoints.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose USI extension defines the metabolomics USI formatting rules and dereferences parsed identifiers via /spectrum/, /png/, /svg/, /json/, /csv/, /qrcode/, /proxi/v0.1/spectra, and /mirror/ endpoints) — github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (HUPO-PSI identifier specification extended for metabolomics; supplies the mzspec field structure being parsed)

## Examples

```
export PYTHONPATH=${PYTHONPATH}:.; pytest -v -n 4 ./test/
```

## Evaluation signals

- Every USI in the README's 'Example USI URLs' section (MS2LDA, MOTIFDB, GNPS task, GNPS library, MassBank, PXD000561, MSV000079514, MSV000078547, MSV000085444, MSV000082791, MSV000082680) appears in the output table
- Each parsed USI has exactly four colon-delimited fields, with empty msRun only for MASSBANK and MOTIFDB accessions
- No collection is misclassified: TASK- prefixed msRun entries map to GNPS/MassIVE task spectra, MSV-prefixed collections map to repository data, CCMSLIB00005436077-class accessions map to GNPS library
- Every USI resolves to a documented backing resource (MASSBANK -> MassBank, MOTIFDB -> MS2LDA, PXD -> ProteomeXchange, MSV -> MassIVE/MetaboLights/Metabolomics Workbench)
- Resolved URLs render successfully against the Metabolomics Spectrum Resolver endpoints (e.g. metabolomics-usi.ucsd.edu/spectrum/?usi=...)

## Limitations

- Identifiers are based on draft USI and draft Metabolomics USI identifiers, so the first block is specified as mzdraft rather than mzspec in some cases (e.g. mzdata:MSV000078547:...) and is subject to change
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories — the metabolomics extension fills this gap but is resolver-specific
- For GNPS library spectra the msRun field is a library name that is ignored, and MassIVE/GNPS repository data follows the external Proteomics USI Standard rather than a metabolomics-specific schema

## Evidence

- [intro] The USI extension routes identifiers to distinct classes of metabolomics data services: [evidence span withheld — anchor status: not_found]
- [readme] Metabolomics USI format for MassBank: "mzspec:MASSBANK::accession:<MassBank Accession>"
- [readme] GNPS task spectrum format: "mzspec:GNPS:TASK-<GNPS Task ID>-<File name in task>:scan:<scan number>"
- [readme] GNPS library spectrum with ignored library name field: "mzspec:GNPS:<GNPS library name (ignored)>:accession:<GNPS Library Accession>"
- [readme] Draft status warning: "These identifiers are based on draft USI and draft Metabolomics USI identifiers."
- [intro] Gap motivating the metabolomics USI extension: [evidence span withheld — anchor status: not_found]
- [methods] Resolver endpoints available for dereferencing: "1. /png/ 1. /svg/ 1. /json/ 1. /proxi/v0.1/spectra 1. /csv/ 1. /qrcode/ 1. /spectrum/ 1. /mirror/"
