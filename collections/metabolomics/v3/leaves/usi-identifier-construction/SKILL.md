---
name: usi-identifier-construction
description: Use when you need a machine-readable, cross-repository identifier for
  an MS/MS spectrum held in a metabolomics or proteomics repository (e.g. MassIVE
  MSV accession, MassBank accession) to fetch or render that spectrum.
when_to_use_negative:
- The spectrum is local (raw vendor file, mzML, MGF) and not deposited in a supported
  repository — no USI can resolve it
- You need a stable, database-independent identifier for publication — SPLASH hashes,
  not USIs, are the database-independent option (though they require a complex hashing
  operation to produce)
- 'You need finalized, stable identifiers for long-term archival: the README warns
  metabolomics USIs are draft (''mzdraft'') and ''subject to change'''
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3344
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that accepts the constructed USI and returns the spectrum as JSON,
    PNG, SVG, CSV, QR code, or mirror plot from /json/, /png/, /svg/, /csv/, /qrcode/,
    /mirror/ endpoints
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:9c2029d9410ecb7130df398617997466219652c29c65fc4b3e2d248bc31c61ac
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# usi-identifier-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs metabolomics-aware Universal Spectrum Identifiers (USI, mzspec:...:/mzdraft:...) for MS/MS spectra stored in MassIVE/GNPS, MetaboLights, Metabolomics Workbench, MassBank, MS2LDA, and ProteomeXchange, so spectra can be retrieved and rendered via the Metabolomics Spectrum Resolver endpoints (/json/, /png/, /svg/, /csv/, /mirror/).

## When to use

Use when you need a machine-readable, cross-repository identifier for an MS/MS spectrum held in a metabolomics or proteomics repository (e.g. MassIVE MSV accession, MassBank accession) to fetch or render that spectrum.

## When NOT to use

- The spectrum is local (raw vendor file, mzML, MGF) and not deposited in a supported repository — no USI can resolve it
- You need a stable, database-independent identifier for publication — SPLASH hashes, not USIs, are the database-independent option (though they require a complex hashing operation to produce)
- You need finalized, stable identifiers for long-term archival: the README warns metabolomics USIs are draft ('mzdraft') and 'subject to change'

## Inputs

- MassIVE/GNPS dataset identifier (MSV accession), source spectrum filename, and scan number
- MetaboLights or Metabolomics Workbench MSV identifier with filename and scan number
- MassBank or GNPS spectral library accession (e.g. CCMSLIB00005436077, SM858102)
- GNPS task ID, task-internal filename, and scan number
- ProteomeXchange PXD identifier with filename and scan number

## Outputs

- A USI string in the form 'mzspec:<repository>:<filename|accession>:scan|accession:<identifier>'
- JSON peak list (m/z and intensity pairs) retrieved via the /json/ endpoint, e.g. MSV000082791_epigallocatechin_scan2_peaks.json

## How to apply

Decide which repository the spectrum lives in, then assemble the USI block by block using the paper's extended metabolomics format: 'mzspec:<repository>:<filename-or-accession>:scan:<scan number>'. For MassIVE/GNPS repository data use the MSV identifier plus the source filename (e.g. 'mzspec:MSV000082791:(-)-epigallocatechin:scan:2'); for library spectra use accession blocks ('mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077', 'mzspec:MASSBANK::accession:SM858102'); for MetaboLights/Metabolomics Workbench use the MSV identifier, filename, and scan number; for GNPS task output use 'mzspec:GNPS:TASK-<task ID>-<filename>:scan:<scan>'. Issue an HTTP GET against the Spectrum Resolver /json/ endpoint with the USI as the 'usi' query parameter, then parse the returned JSON peak list of m/z/intensity pairs. Note the README caveat: these are draft metabolomics identifiers and may be written as 'mzdraft' rather than 'mzspec' until finalized.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that accepts the constructed USI and returns the spectrum as JSON, PNG, SVG, CSV, QR code, or mirror plot from /json/, /png/, /svg/, /csv/, /qrcode/, /mirror/ endpoints) — https://github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
curl "https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:MSV000082791:(-)-epigallocatechin:scan:2"
```

## Evaluation signals

- The USI string follows a documented block pattern exactly (e.g. 'mzspec:<MetaboLights MSV identifier>:<Filename>:scan:<Scan Number>') with the correct scan number and filename spelling
- The HTTP GET to /json/ returns HTTP 200 and the response body parses as valid JSON
- The parsed response contains a non-empty peak list with paired m/z and intensity entries
- The same USI also renders via /svg/ or /png/ without error, confirming the identifier is recognized by the resolver
- Cross-check the resolved spectrum (precursor m/z, peak count) against the source repository entry for the cited scan

## Limitations

- Metabolomics USIs are based on a draft specification; README states they must be marked 'mzdraft' instead of 'mzspec' because identifiers are 'subject to change'
- The USI standard natively supports proteomics data via ProteomeXchange but does not natively support metabolomics repositories, so only the resolver's extended syntax covers MassIVE/MetaboLights/Metabolomics Workbench data
- GNPS library USIs ignore the library-name block ('mzspec:GNPS:<GNPS library name (ignored)>:accession:<GNPS Library Accession>'), which can mislead users expecting it to be meaningful
- The resolver's supported USI types are enumerated (GNPS tasks/libraries, MS2LDA, MassBank, ProteomeXchange, MassIVE, MetaboLights, Metabolomics Workbench); USIs for other sources will not resolve

## Evidence

- [intro] the USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in: [evidence span withheld — anchor status: not_found]
