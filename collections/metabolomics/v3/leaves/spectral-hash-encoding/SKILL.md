---
name: spectral-hash-encoding
description: Use when you have retrieved an MS/MS spectrum (e.g. via a metabolomics-extended
  USI through the Metabolomics Spectrum Resolver) and need an unambiguous, database-independent
  identifier to compare or reference it across resources.
when_to_use_negative:
- When a repository-native identifier (e.g. a GNPS library accession) already suffices
  for within-database referencing and no cross-resource comparison is needed
- When the identifier must conform to the ratified HUPO-PSI USI standard rather than
  a draft metabolomics extension — README warns these identifiers are 'subject to
  change'
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that retrieves spectra by metabolomics-extended USI (via /json/
    endpoint) and computes SPLASH identifiers for the retrieved spectra
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: SPLASH
  role: Spectral hashing algorithm applied to a spectrum's peak list to produce the
    database-independent 'splash10-...' identifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:eb37061887266f8c6be9dbf3f540fd99290523163ffc2add68b1604d15507b0d
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# SPLASH spectral hash encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute SPLASH (SPectraL hASH) identifiers for tandem mass spectra so that spectra can be referenced and compared across metabolomics resources without relying on a database-specific identifier.

## When to use

Use when you have retrieved an MS/MS spectrum (e.g. via a metabolomics-extended USI through the Metabolomics Spectrum Resolver) and need an unambiguous, database-independent identifier to compare or reference it across resources.

## When NOT to use

- When a repository-native identifier (e.g. a GNPS library accession) already suffices for within-database referencing and no cross-resource comparison is needed
- When the identifier must conform to the ratified HUPO-PSI USI standard rather than a draft metabolomics extension — README warns these identifiers are 'subject to change'

## Inputs

- A tandem mass spectrum (peak list of m/z–intensity pairs), e.g. retrieved as JSON from the Metabolomics Spectrum Resolver /json/ endpoint keyed by a metabolomics-extended USI

## Outputs

- A SPLASH identifier following the 'splash10-...' pattern
- A structured record pairing the input USI with the computed SPLASH identifier for cross-resource comparison

## How to apply

Take a spectrum's peak list — for example the machine-readable record returned by the Metabolomics Spectrum Resolver's /json/ endpoint for a metabolomics-extended USI such as mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 — and compute its SPLASH identifier via the SPLASH hashing algorithm. The article notes that SPLASH generation 'includes a complex hashing operation', so spectra should not be hashed ad hoc; the SPLASH identifier follows the 'splash10-...' pattern and serves as an unambiguous, database-independent spectrum identifier. Pair the computed SPLASH with the originating USI in a structured record (e.g. JSON) so the spectrum can be compared across GNPS, MassBank, MassIVE, MetaboLights, or Metabolomics Workbench holdings regardless of each repository's native identifier scheme.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that retrieves spectra by metabolomics-extended USI (via /json/ endpoint) and computes SPLASH identifiers for the retrieved spectra) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **SPLASH** (Spectral hashing algorithm applied to a spectrum's peak list to produce the database-independent 'splash10-...' identifier)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077' | python -c "import json,sys; s=json.load(sys.stdin); print(s['splash'])"
```

## Evaluation signals

- The computed identifier matches the SPLASH pattern 'splash10-...'
- The output record contains both the originating USI (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:...) and the SPLASH fields
- Two spectra with identical peak lists yield identical SPLASH identifiers, independent of the repository they were retrieved from
- The identifier is database-independent: it does not embed the source resource name (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench)

## Limitations

- SPLASH identifiers cannot be trivially produced because the algorithm involves a complex, non-trivial hashing operation
- The metabolomics-extended USIs used to retrieve the spectra are based on draft standards ('mzdraft') and are subject to change per the README

## Evidence
