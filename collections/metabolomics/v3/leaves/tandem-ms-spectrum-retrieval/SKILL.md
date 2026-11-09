---
name: tandem-ms-spectrum-retrieval
description: Use when you need to pull a specific MS/MS spectrum out of a public resource
  (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA) without
  downloading the repository itself, and you know its source, accession, or scan number.
when_to_use_negative:
- You need a database-independent spectrum identifier for comparison — use SPLASH
  hashing instead, since SPLASH identifiers are designed to be unambiguous and database-independent
- The spectrum exists only as raw instrument data not yet deposited in a supported
  repository (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
- You already hold the spectrum locally (e.g. an MGF or mzML file) and just need plotting
  or annotation — use the resolver's rendering endpoints directly rather than retrieval
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves the USI against the source repository and returns
    the spectrum in human- and machine-readable formats (JSON, CSV, SVG, PNG, QR code)
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: Standardized identifier grammar used to encode which repository, file, and
    spectrum to retrieve, extended here for metabolomics use cases
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:8d68571c411e09cf6526f6990c70e18a0c1be87d9042a35175e34e4d782758df
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tandem-ms-spectrum-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve a tandem mass spectrometry spectrum from public metabolomics/proteomics repositories by constructing a metabolomics-extended Universal Spectrum Identifier (USI, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077) and querying the Metabolomics Spectrum Resolver's machine-readable endpoints (/json/, /csv/, PROXI).

## When to use

Use when you need to pull a specific MS/MS spectrum out of a public resource (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA) without downloading the repository itself, and you know its source, accession, or scan number.

## When NOT to use

- You need a database-independent spectrum identifier for comparison — use SPLASH hashing instead, since SPLASH identifiers are designed to be unambiguous and database-independent
- The spectrum exists only as raw instrument data not yet deposited in a supported repository (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
- You already hold the spectrum locally (e.g. an MGF or mzML file) and just need plotting or annotation — use the resolver's rendering endpoints directly rather than retrieval

## Inputs

- Metabolomics-extended USI string (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077)
- Source repository designation (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
- Accession number or scan number identifying the spectrum within its source
- Optional filename for repository-data USIs (e.g. Adult_Frontalcortex_bRP_Elite_85_f09)

## Outputs

- Machine-readable spectrum (peak list JSON) from /json/
- PROXI-format spectrum response from /proxi/v0.1/spectra
- CSV tabular peak list from /csv/
- Rendered spectrum images (SVG/PNG) with configurable parameters

## How to apply

First determine which of the supported USI types applies — GNPS library spectrum, GNPS/MassIVE analysis task spectrum, MassBank library spectrum, MS2LDA/MOTIFDB motif, or repository data from ProteomeXchange, MassIVE, MetaboLights, or Metabolomics Workbench — because each has its own mzspec: grammar (e.g. mzspec:GNPS:<library name>:accession:<accession> vs mzspec:<MSV identifier>:<filename>:scan:<scan number>). Construct the metabolomics-extended USI accordingly; the draft metabolomics identifiers may be prefixed mzdraft instead of mzspec. Query the Metabolomics Spectrum Resolver (https://metabolomics-usi.ucsd.edu) with the USI: use /json/ for a machine-readable spectrum, /csv/ for tabular output, /proxi/v0.1/spectra for PROXI-format retrieval, and /spectrum/ for the interactive view. Optionally constrain retrieval to a peak-list subset via m/z range parameters (mz_min, mz_max). The rationale is that no standardized spectrum identifier exists for metabolomics data, so the USI encodes resource, file, and scan/accession in one addressable string that the resolver translates into an actual spectrum lookup across heterogeneous repository formats.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves the USI against the source repository and returns the spectrum in human- and machine-readable formats (JSON, CSV, SVG, PNG, QR code)) — github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (Standardized identifier grammar used to encode which repository, file, and spectrum to retrieve, extended here for metabolomics use cases)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077'
```

## Evaluation signals

- The USI parses without error against the resolver for the given USI type (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:... vs mzspec:MassIVE:<MSV id>:<filename>:scan:<n>)
- /json/ endpoint returns a spectrum object with a peak list matching the requested scan/accession
- The requested mzspec/mzdraft prefix is consistent with whether the identifier is draft metabolomics USI (subject to change) or standard USI
- PROXI /proxi/v0.1/spectra returns a response for the same USI, confirming cross-endpoint consistency
- Retrieved spectrum's precursor m/z and peak list are consistent when re-queried via /csv/ or /spectrum/

## Limitations

- These identifiers are based on draft USI and draft Metabolomics USI identifiers and are subject to change; they are specified as mzdraft instead of mzspec in the first block
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support metabolomics repositories, hence the metabolomics extension
- GNPS library USIs ignore the library-name field (mzspec:GNPS:<GNPS library name (ignored)>:accession:<accession>)

## Evidence

- [intro] Motivation for USI-based retrieval: [evidence span withheld — anchor status: not_found]
