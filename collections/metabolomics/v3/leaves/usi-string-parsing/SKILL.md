---
name: usi-string-parsing
description: Use when a task requires parsing mzspec:<collection>:<msRun>:<indexType>:<indexNumber>
  strings from GNPS, MassBank, MS2LDA/MOTIFDB, MassIVE, MetaboLights, Metabolomics
  Workbench, or ProteomeXchange into fields and mapping each to its backing resource.
when_to_use_negative:
- The identifier is a SPLASH hash — the paper notes SPLASH generation requires a complex
  hashing operation and is a different, database-independent scheme, not a colon-delimited
  USI to parse.
- The USI is a proteomics-specific construct outside the metabolomics extension's
  supported sources — the standard 'natively' supports ProteomeXchange but the resolver
  only handles the listed metabolomics collection types.
- The goal is spectrum rendering or figure styling rather than parsing/routing — use
  the resolver's /png/, /svg/, /json/, /csv/ endpoints with plotting parameters (mz_min,
  mz_max, annotate_peaks) instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Reference implementation that parses metabolomics-extended USIs and routes
    them to data repositories (MassIVE, MetaboLights, Metabolomics Workbench) and
    reference spectral libraries (MassBank, MoNA/GNPS); provides USI URL endpoints
    for spectrum retrieval.
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: The HUPO-PSI identifier standard being parsed; the metabolomics extension
    defines the collection-specific formatting rules applied here.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:1d061fe7e80565dd80546263a58c6925ed1e7818c954be87bcb30392c12cdc66
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# usi-string-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Decompose metabolomics Universal Spectrum Identifiers (USIs) into their four colon-delimited fields (collection, msRun, indexType, indexNumber) and route each identifier to the correct backing data service (repository or spectral library) as implemented in the Metabolomics Spectrum Resolver.

## When to use

Use when a task requires parsing mzspec:<collection>:<msRun>:<indexType>:<indexNumber> strings from GNPS, MassBank, MS2LDA/MOTIFDB, MassIVE, MetaboLights, Metabolomics Workbench, or ProteomeXchange into fields and mapping each to its backing resource.

## When NOT to use

- The identifier is a SPLASH hash — the paper notes SPLASH generation requires a complex hashing operation and is a different, database-independent scheme, not a colon-delimited USI to parse.
- The USI is a proteomics-specific construct outside the metabolomics extension's supported sources — the standard 'natively' supports ProteomeXchange but the resolver only handles the listed metabolomics collection types.
- The goal is spectrum rendering or figure styling rather than parsing/routing — use the resolver's /png/, /svg/, /json/, /csv/ endpoints with plotting parameters (mz_min, mz_max, annotate_peaks) instead.

## Inputs

- USI strings in the form mzspec:<collection>:<msRun>:<indexType>:<indexNumber> (e.g. mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943)
- Metabolomics USI draft variants using the mzdraft prefix (e.g. mzdata:MSV000078547:120228_nbut_3610_it_it_take2:scan:389)

## Outputs

- Structured table (e.g. usi_collection_map.csv) with parsed fields (collection, msRun, indexType, indexNumber) and resolved backing resource per USI
- Collection-type classification per USI (MassBank library spectrum, GNPS task spectrum, MS2LDA MOTIFDB, GNPS library spectrum, MetaboLights/Metabolomics Workbench/MassIVE repository spectrum)

## How to apply

Split each USI on colons into collection, msRun, indexType, and indexNumber, handling the documented special cases: the msRun field is empty for MassBank (mzspec:MASSBANK::accession:<accession>) and MOTIFDB (mzspec:MOTIFDB::accession:<accession>) entries, and is an ignored library name for GNPS library spectra (mzspec:GNPS:<library name>:accession:CCMSLIB...). Classify each parsed USI by collection type using the documented formatting patterns: 'TASK-<GNPS Task ID>-<File name>' marks GNPS/MassIVE analysis task spectra (with an optional '/specs_ms.mgf' file suffix distinguishing file-level from collection-level task spectra), PXD* accessions route to ProteomeXchange, MSV* identifiers route to MassIVE/GNPS, MetaboLights, or Metabolomics Workbench repository data. Map each collection type to its resolved backing resource (MASSBANK → MassBank, MOTIFDB → MS2LDA, MSV/PXD → repository services). Emit one row per input USI with its parsed components and resolved resource, covering every identifier in the methods' Example USI URLs section. Note the README caveat that these identifiers are draft ('mzdraft') and subject to change.

## Related tools

- **Metabolomics Spectrum Resolver** (Reference implementation that parses metabolomics-extended USIs and routes them to data repositories (MassIVE, MetaboLights, Metabolomics Workbench) and reference spectral libraries (MassBank, MoNA/GNPS); provides USI URL endpoints for spectrum retrieval.) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (The HUPO-PSI identifier standard being parsed; the metabolomics extension defines the collection-specific formatting rules applied here.)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra:scan:1943'
```

## Evaluation signals

- Every USI in the README's Example USI URLs section appears in the output table with a non-empty four-field decomposition.
- Collection classification matches the documented formatting patterns: MASSBANK:: and MOTIFDB:: entries have an empty msRun field; GNPS task spectra start with 'TASK-'; MSV/PXD identifiers route to repository resources.
- Each resolved backing resource is one of the documented services (MassBank, MS2LDA, ProteomeXchange, MassIVE, MetaboLights, Metabolomics Workbench, GNPS).
- Special-case handling verified: mzspec:MASSBANK::accession:SM858102 parses with an empty msRun field, and mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 ignores the GNPS-LIBRARY library-name field.
- The resolver accepts each parsed USI at a rendering endpoint (e.g. /svg/?usi=...) without an unresolved-collection error.

## Limitations

- These identifiers are based on draft USI and draft Metabolomics USI identifiers, 'subject to change', and specified as mzdraft rather than mzspec in the first block.
- The USI standard 'currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories' — hence the extension's custom collection types.
- One README example (MassIVE/GNPS MS1 spectra, mzspec:MSV000078547:...) uses the mzdata prefix, showing prefix inconsistency in the draft spec.
- For GNPS library spectra the library-name msRun field is ignored during routing, so it carries no distinguishing information.

## Evidence

- [intro] The USI extension routes identifiers to distinct classes of metabolomics data services: data repositories (MassIVE, MetaboLights, Metabolomics Workbench) and reference spectral library resources (MassBank, MoNA), alongside online informatics pipelines.: [evidence span withheld — anchor status: not_found]
- [intro] USI standard does not natively support metabolomics repositories, motivating the extension.: [evidence span withheld — anchor status: not_found]
