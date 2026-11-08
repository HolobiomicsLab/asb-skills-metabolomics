---
name: splash-identifier-computation
description: Use when an MS/MS spectrum has been retrieved via a metabolomics USI
  (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:...) and it must be compared across public
  resources, but no standardized spectrum identifier is available.
when_to_use_negative:
- The goal is only to retrieve or visualize a spectrum (PNG/SVG/mirror plots), not
  to obtain a database-independent cross-resource identifier
- The spectrum's peak list is unavailable or cannot be fetched via the /json/ endpoint,
  since SPLASH computation requires the peak list
- A plain proteomics USI lookup via ProteomeXchange is sufficient — USIs are easy
  to construct for proteomics data deposited via ProteomeXchange without needing SPLASH
  hashing
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Metabolomics Spectrum Resolver
  role: Provides the /json/ endpoint that returns the machine-readable spectrum (peak
    list) for a metabolomics-extended USI, which is the input to SPLASH computation
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: SPLASH
  role: Hashing algorithm applied to the retrieved spectrum's peak list to produce
    the database-independent splash10-... identifier
- name: USI
  role: Universal Spectrum Identifier specification, extended for metabolomics use
    cases, used as the retrieval key and paired in the output record
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:c7dc52bf9612d159a6dd1927031fc167e2b2b71f5b8d328a037074b6b48cd690
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# splash-identifier-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes SPLASH (SPectraL hASH) identifiers from the peak lists of MS/MS spectra retrieved via the Metabolomics Spectrum Resolver's /json/ endpoint, yielding an unambiguous, database-independent spectrum identifier that can be paired with the input metabolomics-extended USI for cross-resource comparison.

## When to use

Use when an MS/MS spectrum has been retrieved via a metabolomics USI (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:...) and it must be compared across public resources, but no standardized spectrum identifier is available.

## When NOT to use

- The goal is only to retrieve or visualize a spectrum (PNG/SVG/mirror plots), not to obtain a database-independent cross-resource identifier
- The spectrum's peak list is unavailable or cannot be fetched via the /json/ endpoint, since SPLASH computation requires the peak list
- A plain proteomics USI lookup via ProteomeXchange is sufficient — USIs are easy to construct for proteomics data deposited via ProteomeXchange without needing SPLASH hashing

## Inputs

- Machine-readable MS/MS spectrum (peak list) retrieved from the Metabolomics Spectrum Resolver /json/ endpoint, keyed by a metabolomics-extended USI such as mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077
- USI string (mzspec:... format) identifying the source repository and accession

## Outputs

- SPLASH identifier string following the 'splash10-...' pattern computed from the spectrum's peak list
- Structured JSON record pairing the input USI with the computed SPLASH identifier

## How to apply

Load the machine-readable spectrum (peak list) from the Spectrum Resolver's /json/ endpoint using the metabolomics-extended USI, since SPLASH generation involves a complex hashing operation that cannot be trivially produced from a peak list by hand. Compute the SPLASH identifier from the spectrum's peak list using the SPLASH hashing algorithm. Assemble a structured JSON record pairing the input USI with the computed SPLASH identifier so the spectrum can be unambiguously compared across repositories such as GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench. Validate the record is well-formed: SPLASH identifiers follow the 'splash10-...' pattern and the record must contain both the USI and SPLASH fields.

## Related tools

- **Metabolomics Spectrum Resolver** (Provides the /json/ endpoint that returns the machine-readable spectrum (peak list) for a metabolomics-extended USI, which is the input to SPLASH computation) — github.com/mwang87/MetabolomicsSpectrumResolver
- **SPLASH** (Hashing algorithm applied to the retrieved spectrum's peak list to produce the database-independent splash10-... identifier)
- **USI** (Universal Spectrum Identifier specification, extended for metabolomics use cases, used as the retrieval key and paired in the output record)

## Examples

```
curl https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077
```

## Evaluation signals

- Computed SPLASH matches the 'splash10-...' pattern (four hyphen-delimited segments after splash10-)
- Output record contains both the input USI (e.g. mzspec:GNPS:...) and the computed SPLASH fields
- Identical peak lists submitted from different repositories (e.g. GNPS vs MassBank) yield the same SPLASH, demonstrating database independence
- Re-running the computation on the same /json/ spectrum is deterministic and reproduces the same SPLASH identifier
- The USI resolves successfully at https://metabolomics-usi.ucsd.edu/ before hashing, confirming the retrieved spectrum is valid

## Limitations

- SPLASH generation involves a complex hashing operation, so identifiers cannot be trivially produced from spectra directly
- The metabolomics USI specification is based on draft identifiers and may be subject to change (README notes identifiers may need to be specified as 'mzdraft' instead of 'mzspec')
- The USI standard natively supports proteomics spectral data via ProteomeXchange; metabolomics repositories require the extended mzspec format

## Evidence
