---
name: cross-resource-spectrum-comparison
description: Use when spectra from different metabolomics/proteomics resources must
  be compared or linked unambiguously and no standardized cross-repository spectrum
  identifier exists; e.g. pairing a metabolomics-extended USI (mzspec:...) with its
  database-independent SPLASH.
when_to_use_negative:
- The spectrum is already addressed by a shared, native identifier within a single
  resource — a SPLASH computation adds nothing for intra-repository lookup.
- The goal is proteomics-only spectral retrieval via ProteomeXchange; plain USI support
  suffices without metabolomics extension or SPLASH hashing.
- Only a visualization/export is needed (PNG/SVG/mirror plot); the /png/, /svg/ or
  /mirror/ endpoints should be used instead of computing identifiers.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service used to retrieve the machine-readable spectrum by USI via the
    /json/ endpoint before SPLASH computation
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: SPLASH
  role: Hashing algorithm applied to the spectrum's peak list to produce the database-independent
    spectrum identifier
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier, extended for metabolomics, used as
    the retrieval key and paired with the SPLASH in the output record
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:341879f9f22b4059a7543a59ceefd67094dc1cfee39835025f43ef112b242561
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
  - SPLASH
  - USI
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-resource-spectrum-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes SPLASH (SPectraL hASH) identifiers from tandem MS peak lists so spectra retrieved from different public resources (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench) can be compared without a shared database identifier.

## When to use

Use when spectra from different metabolomics/proteomics resources must be compared or linked unambiguously and no standardized cross-repository spectrum identifier exists; e.g. pairing a metabolomics-extended USI (mzspec:...) with its database-independent SPLASH.

## When NOT to use

- The spectrum is already addressed by a shared, native identifier within a single resource — a SPLASH computation adds nothing for intra-repository lookup.
- The goal is proteomics-only spectral retrieval via ProteomeXchange; plain USI support suffices without metabolomics extension or SPLASH hashing.
- Only a visualization/export is needed (PNG/SVG/mirror plot); the /png/, /svg/ or /mirror/ endpoints should be used instead of computing identifiers.

## Inputs

- Machine-readable spectrum JSON from the Metabolomics Spectrum Resolver /json/ endpoint
- Metabolomics-extended USI string (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077, mzspec:MASSBANK::accession:SM858102, mzspec:MSV000082680:iPSC-T1R1:scan:3)
- Tandem MS/MS peak list (m/z, intensity pairs) underlying the spectrum

## Outputs

- SPLASH identifier string following the 'splash10-...' pattern
- Structured JSON record pairing the input USI with its computed SPLASH identifier

## How to apply

Retrieve the machine-readable spectrum from the Metabolomics Spectrum Resolver via its /json/ endpoint, addressing it by a metabolomics-extended USI (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077). Extract the spectrum's peak list and compute the SPLASH identifier using the SPLASH hashing algorithm — the paper notes that generating a SPLASH identifier involves a complex, non-trivial hashing operation, so it cannot be produced by hand. Assemble a structured JSON record that pairs the input USI with the computed SPLASH, since USIs are easy to construct but (as drafted) do not natively cover metabolomics repositories, whereas SPLASH is unambiguous and database-independent. Repeat for each spectrum from each resource; matching SPLASH values then establish equivalence across resources. Note the README caveat that these identifiers were based on draft USI/Metabolomics-USI specifications (mzdraft vs mzspec) and are subject to change.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service used to retrieve the machine-readable spectrum by USI via the /json/ endpoint before SPLASH computation) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **SPLASH** (Hashing algorithm applied to the spectrum's peak list to produce the database-independent spectrum identifier)
- **USI** (HUPO-PSI Universal Spectrum Identifier, extended for metabolomics, used as the retrieval key and paired with the SPLASH in the output record)

## Examples

```
curl https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077
```

## Evaluation signals

- Computed SPLASH matches the 'splash10-...' well-formedness pattern.
- The output record contains both the 'usi' and 'splash' fields populated.
- Two spectra known to be identical (e.g. the same library spectrum retrieved from two resources) yield the same SPLASH identifier.
- Re-running the hash on the same peak list is deterministic and reproduces the identical SPLASH value.
- Identifiers differing at any peak m/z or intensity produce distinct SPLASH values (no collisions on trivially perturbed spectra).

## Limitations

- SPLASH generation involves a complex hashing operation and cannot be trivially produced; a library/implementation of the algorithm is required.
- The metabolomics USI extension was draft at the time (README warns identifiers 'will be specified as mzdraft instead of mzspec') and is subject to change.
- The USI standard mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories.

## Evidence

- [intro] The SPectraL hASH (SPLASH) identifier has been designed as an unambiguous, database-independent spectrum identifier: "The SPectraL hASH (SPLASH) identifier has been designed as an unambiguous, database-independent spectrum identifier"
- [intro] as generating a SPLASH identifier includes a complex hashing operation, these identifiers cannot be trivially produced: "as generating a SPLASH identifier includes a complex hashing operation, these identifiers cannot be trivially produced."
- [intro] the USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories.: [evidence span withheld — anchor status: not_found]
- [readme] These identifiers are based on draft USI and draft Metabolomics USI identifiers. Thus, they are subject to change: "These identifiers are based on draft USI and draft Metabolomics USI identifiers. 
        Thus, they are subject to change"
- [abstract] this resource already supports data formats from GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench: "this resource already supports data formats from GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench"
