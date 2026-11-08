---
name: mass-spectrometry-metadata-extraction
description: 'Use when input contains mzspec:/mzdraft: USI strings that must be split
  into collection, msRun, indexType, indexNumber fields and mapped to the correct
  metabolomics data repository or spectral library service.'
when_to_use_negative:
- The spectrum identifier is a SPLASH hash rather than a USI — SPLASH generation involves
  a complex hashing operation and follows a different identification scheme
- The goal is interactive spectral visualization or figure customization (annotation,
  m/z range, mirror plots) rather than parsing identifiers and routing them to data
  services
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose USI extension defines the collection-type classification
    and backing-resource routing implemented by this skill; exposes endpoints such
    as /png/, /svg/, /json/, /csv/, /proxi/v0.1/spectra and /mirror/ that consume
    the resolved USIs
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier specification extended for metabolomics
    use cases; provides the mzspec colon-delimited field structure being parsed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:f43565c09c26b860500581fd35332697300c8d4c198e90c6d4ec60ebbfc84a47
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-metadata-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Decomposes metabolomics-flavored Universal Spectrum Identifiers (USI/mzspec strings) into their colon-delimited fields (collection, msRun, indexType, indexNumber) and routes each identifier to its backing resource (MassIVE, MetaboLights, Metabolomics Workbench, MassBank, MS2LDA/MOTIFDB, GNPS). This is the core mechanism the Metabolomics Spectrum Resolver uses to fetch and render spectra from heterogeneous repositories.

## When to use

Use when input contains mzspec:/mzdraft: USI strings that must be split into collection, msRun, indexType, indexNumber fields and mapped to the correct metabolomics data repository or spectral library service.

## When NOT to use

- The spectrum identifier is a SPLASH hash rather than a USI — SPLASH generation involves a complex hashing operation and follows a different identification scheme
- The goal is interactive spectral visualization or figure customization (annotation, m/z range, mirror plots) rather than parsing identifiers and routing them to data services

## Inputs

- USI strings in the form mzspec:<collection>:<msRun>:<indexType>:<indexNumber> (e.g. from GNPS tasks, GNPS library spectra, MS2LDA MOTIFDB, MassBank, ProteomeXchange PXD, MassIVE MSV, MetaboLights, Metabolomics Workbench)
- Methods documentation 'Example USI URLs' section enumerating the identifiers to cover

## Outputs

- usi_collection_map.csv: one row per input USI with parsed fields (collection, msRun, indexType, indexNumber) and the resolved backing resource

## How to apply

Collect the USI strings and split each on colons into the four fields mzspec:<collection>:<msRun>:<indexType>:<indexNumber>, handling the documented special cases: an empty msRun field for MassBank and MOTIFDB accessions (e.g. mzspec:MASSBANK::accession:SM858102) and a GNPS library-name msRun field that is ignored for GNPS library spectra (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077). Classify each parsed USI by collection type using the formatting patterns: MassBank library spectra (mzspec:MASSBANK::accession:<accession>), GNPS analysis task spectra (mzspec:GNPS:TASK-<task ID>-<file name>:scan:<scan>), MS2LDA MOTIFDB entries (mzspec:MOTIFDB::accession:<accession>), GNPS library spectra, and the MetaboLights / Metabolomics Workbench / MassIVE MSV-based repository schemas. Then map each collection type to its backing resource: MASSBANK to MassBank, MOTIFDB to MS2LDA, PXD accessions to ProteomeXchange, MSV accessions to MassIVE/MetaboLights/Metabolomics Workbench, consistent with the Metabolomics Spectrum Resolver's USI extension. Finally emit the parsed components and resolved resource for every input USI as rows of the structured table usi_collection_map.csv.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose USI extension defines the collection-type classification and backing-resource routing implemented by this skill; exposes endpoints such as /png/, /svg/, /json/, /csv/, /proxi/v0.1/spectra and /mirror/ that consume the resolved USIs) — github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (HUPO-PSI Universal Spectrum Identifier specification extended for metabolomics use cases; provides the mzspec colon-delimited field structure being parsed)

## Examples

```
usi="mzspec:MASSBANK::accession:SM858102"; collection, msRun, indexType, indexNumber = usi.split(":")  # msRun is empty for MASSBANK/MOTIFDB
```

## Evaluation signals

- Every USI listed in the methods' 'Example USI URLs' section appears in usi_collection_map.csv and is classified per the documented formatting patterns
- Special cases parsed correctly: empty msRun field for MASSBANK and MOTIFDB accessions; GNPS library-name msRun field ignored (GNPS-LIBRARY)
- Each parsed USI resolves to the documented backing resource (PXD→ProteomeXchange, MSV→MassIVE/MetaboLights/Metabolomics Workbench, MASSBANK→MassBank, MOTIFDB→MS2LDA); any misclassified collection or unresolved resource fails
- Index type/value pairs preserved verbatim (e.g. scan:1943, accession:43062, accession:CCMSLIB00005436077)
- Repository unit tests pass (pytest -v -n 4 ./test/ with PYTHONPATH set), per the project's unittest workflow

## Limitations

- Identifiers are based on draft USI and draft Metabolomics USI identifiers, so the leading block may be mzdraft instead of mzspec and the format is subject to change
- The standard USI specification natively supports mainly proteomics data deposited via ProteomeXchange; metabolomics repositories are covered only by the extension implemented here
- The msRun field carries collection-specific semantics (empty for MassBank/MOTIFDB, an ignored library name for GNPS library spectra), so naive positional parsing without these special cases misroutes identifiers
- No changelog or software version is stated for the Metabolomics Spectrum Resolver implementation

## Evidence

- [intro] the USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in: [evidence span withheld — anchor status: not_found]
- [intro] Data repositories: MassIVE, MetaboLights, Metabolomics Workbench.: [evidence span withheld — anchor status: not_found]
