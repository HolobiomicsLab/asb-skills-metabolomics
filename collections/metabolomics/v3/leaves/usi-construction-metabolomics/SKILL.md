---
name: usi-construction-metabolomics
description: Use when a spectrum must be addressed across public repositories (GNPS
  library/task, MassBank accession, MassIVE/MetaboLights/Workbench MSV identifiers)
  or compared between resources via a database-independent identifier.
when_to_use_negative:
- The spectrum is already referenced by a native identifier that the target repository
  resolves directly (e.g. a ProteomeXchange PXD dataset under the standard proteomics
  USI specification) and no cross-resource comparison is needed
- The goal is spectral rendering rather than identification/comparison — use the /png/
  or /svg/ endpoints instead of computing identifiers
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose /json/ endpoint retrieves the machine-readable spectrum
    keyed by the metabolomics-extended USI prior to SPLASH computation
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: SPLASH
  role: Hashing algorithm applied to the spectrum's peak list to produce the database-independent
    SPLASH identifier
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier specification, extended for metabolomics
    use cases, used to address spectra from repositories not natively supported by
    the standard
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:7e74ba53286c00b73e86b03d92a448574135e3f1ce3ae922e55f91fc56788782
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
  - SPLASH
  - USI
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# usi-construction-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs metabolomics-extended Universal Spectrum Identifiers (USIs, 'mzspec:...') for spectra from GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA and ProteomeXchange, and computes SPLASH identifiers as database-independent spectrum identifiers. It enables unambiguous cross-resource spectrum comparison despite the lack of a native standardized identifier in metabolomics repositories.

## When to use

Use when a spectrum must be addressed across public repositories (GNPS library/task, MassBank accession, MassIVE/MetaboLights/Workbench MSV identifiers) or compared between resources via a database-independent identifier.

## When NOT to use

- The spectrum is already referenced by a native identifier that the target repository resolves directly (e.g. a ProteomeXchange PXD dataset under the standard proteomics USI specification) and no cross-resource comparison is needed
- The goal is spectral rendering rather than identification/comparison — use the /png/ or /svg/ endpoints instead of computing identifiers

## Inputs

- Metabolomics-extended USI string (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077) for a GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB, or ProteomeXchange spectrum
- Machine-readable spectrum peak list (e.g. from the resolver's /json/ endpoint)

## Outputs

- Well-formed SPLASH identifier following the 'splash10-...' pattern
- Structured JSON record pairing the input USI with its computed SPLASH identifier

## How to apply

First determine the spectrum's origin and assemble the matching metabolomics-extended USI pattern from the README, e.g. 'mzspec:GNPS:<library name>:accession:<CCMSLIB accession>' for GNPS library spectra, 'mzspec:MASSBANK::accession:<MassBank accession>' for MassBank, 'mzspec:<MSV identifier>:<filename>:scan:<scan number>' for MetaboLights/Metabolomics Workbench, and the Proteomics USI standard for ProteomeXchange/MassIVE repository data. Retrieve the machine-readable spectrum from the Metabolomics Spectrum Resolver's /json/ endpoint keyed by this USI (use 'mzdraft' instead of 'mzspec' if working with draft identifiers). Then compute the SPLASH identifier from the spectrum's peak list using the SPLASH hashing algorithm — this is a complex, non-trivial hashing operation and cannot be trivially produced by hand. Finally assemble a JSON record pairing the input USI with the computed SPLASH (pattern 'splash10-...') to enable unambiguous, database-independent comparison across resources.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose /json/ endpoint retrieves the machine-readable spectrum keyed by the metabolomics-extended USI prior to SPLASH computation) — github.com/mwang87/MetabolomicsSpectrumResolver
- **SPLASH** (Hashing algorithm applied to the spectrum's peak list to produce the database-independent SPLASH identifier)
- **USI** (HUPO-PSI Universal Spectrum Identifier specification, extended for metabolomics use cases, used to address spectra from repositories not natively supported by the standard)

## Examples

```
curl https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077
```

## Evaluation signals

- Computed identifier matches the 'splash10-...' SPLASH pattern
- The output record contains both the 'USI' and 'SPLASH' fields populated
- Identical peak lists from different repositories (e.g. the same spectrum in GNPS and MassBank) yield the same SPLASH identifier, confirming database independence
- The USI resolves successfully through the resolver's /json/ endpoint, confirming it matches one of the seven supported USI types
- For draft identifiers, the first block uses 'mzdraft' rather than 'mzspec' as the README warns identifiers are subject to change

## Limitations

- SPLASH generation involves a complex hashing operation, so identifiers cannot be trivially produced without implementing the algorithm
- The metabolomics USI extensions are based on draft USI and draft Metabolomics USI identifiers, are subject to change, and may need the 'mzdraft' prefix instead of 'mzspec'
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories, hence the extensions

## Evidence
