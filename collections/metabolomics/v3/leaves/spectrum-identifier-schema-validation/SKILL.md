---
name: spectrum-identifier-schema-validation
description: Decision support for identifiers presented as mzspec/mzdraft colon-delimited
  strings whose collection segment must be matched against known metabolomics and
  proteomics resources (GNPS, MassIVE, MetaboLights, Metabolomics Workbench, MassBank,
  MS2LDA) before spectra can be retrieved.
when_to_use_negative:
- The identifier is a SPLASH hash rather than a colon-delimited USI — SPLASH involves
  a complex hashing operation and is not parseable by these USI formatting rules.
- The spectra are already locally loaded (e.g. MGF files on disk) and no identifier-based
  remote retrieval is needed.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that validates the USI schema, classifies the collection, and
    dispatches the identifier to the correct backing metabolomics data service.
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:3b8d389d6833606df5d34a5e06f9d0fec0d3ca971f9e46fc13fcbf77ce511414
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  repo_url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - USI
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spectrum-identifier-schema-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate and parse Universal Spectrum Identifiers (USIs) as implemented in the Metabolomics Spectrum Resolver, checking the mzspec:<collection>:<msRun>:<indexType>:<indexNumber> schema and resolving each identifier to its backing metabolomics data service.

## When to use

Decision support for identifiers presented as mzspec/mzdraft colon-delimited strings whose collection segment must be matched against known metabolomics and proteomics resources (GNPS, MassIVE, MetaboLights, Metabolomics Workbench, MassBank, MS2LDA) before spectra can be retrieved.

## When NOT to use

- The identifier is a SPLASH hash rather than a colon-delimited USI — SPLASH involves a complex hashing operation and is not parseable by these USI formatting rules.
- The spectra are already locally loaded (e.g. MGF files on disk) and no identifier-based remote retrieval is needed.

## Inputs

- USI strings such as 'mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077', 'mzspec:MSV000082791:(-)-epigallocatechin:scan:2', 'mzspec:MOTIFDB::accession:171163'
- mzdraft identifiers flagged as draft Metabolomics USIs

## Outputs

- Parsed tuple (collection, msRun, indexType, indexNumber) per identifier
- Mapping of each USI to its backing resource (MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
- Retrieval-ready URLs for endpoints such as /png/, /svg/, /json/, /csv/

## How to apply

1. Split the identifier into four colon-delimited fields: collection, msRun, indexType, and indexNumber; note that MassBank and MOTIFDB identifiers leave the msRun field empty (e.g. 'mzspec:MASSBANK::accession:SM858102'), and GNPS library spectra embed a library name that is ignored. 2. Classify each USI by collection using the documented formatting rules — 'mzspec:MASSBANK::accession:<accession>' for MassBank, 'mzspec:GNPS:TASK-<id>-<file>:scan:<n>' for GNPS task spectra, 'mzspec:MOTIFDB::accession:<accession>' for MS2LDA motifs, 'mzspec:GNPS:<library>:accession:<accession>' for GNPS library spectra, and 'mzspec:<MSV/PXD identifier>:<filename>:scan:<scan>' for repository data. 3. Map the collection to its backing resource: MASSBANK to MassBank, MOTIFDB to MS2LDA, PXD accessions to ProteomeXchange, MSV accessions to MassIVE/MetaboLights/Metabolomics Workbench. 4. Reject or flag identifiers that begin with 'mzdraft' as draft Metabolomics USIs whose format may change. 5. Emit each parsed USI with its collection type and resolved backing resource so retrieval endpoints (/png/, /svg/, /json/, /csv/) can be constructed. Correct application is judged by whether every USI parses into exactly four fields and whether the resolved resource matches the collection prefix rules.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that validates the USI schema, classifies the collection, and dispatches the identifier to the correct backing metabolomics data service.) — https://github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
python -c "usi='mzspec:MASSBANK::accession:SM858102'; parts=usi.split(':'); assert parts[2]=='accession' and parts[1]==''"
```

## Evaluation signals

- Every USI in the input set parses into exactly four colon-delimited fields without exception (including empty msRun for MASSBANK and MOTIFDB).
- Each USI is assigned a collection type that matches one of the documented formatting patterns (MassBank, GNPS task, MS2LDA MOTIFDB, GNPS library, MetaboLights, Metabolomics Workbench, MassIVE).
- The resolved backing resource (MassBank, MS2LDA, ProteomeXchange, MassIVE, MetaboLights, Metabolomics Workbench) is consistent with the collection prefix (MASSBANK, MOTIFDB, PXD, MSV).
- Identifiers beginning with 'mzdraft' are flagged as draft Metabolomics USIs subject to change rather than treated as finalized mzspec identifiers.
- Generated retrieval URLs match the examples in the README, e.g. 'mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943'.

## Limitations

- Identifiers are based on draft USI and draft Metabolomics USI identifiers and may change; the prefix is 'mzdraft' instead of 'mzspec' during the draft phase.
- The USI standard natively supports proteomics data deposited via ProteomeXchange and requires the metabolomics extension documented here to handle metabolomics repositories.
- For GNPS library spectra the msRun field contains a library name that is ignored, so collection classification must not rely on that field alone.

## Evidence

- [intro] USI scope for proteomics: [evidence span withheld — anchor status: not_found]
