---
name: metabolomics-repository-data-access
description: Use when a spectrum of interest lives in a metabolomics repository (e.
when_to_use_negative:
- You need unambiguous, database-independent spectrum identification via SPLASH —
  SPLASH requires a complex hashing operation and cannot be trivially produced, unlike
  USIs.
- The spectrum is already in hand (local mzML/MGF peak list) and only needs rendering/annotation
  customization — direct spectrum_utils plotting avoids network round-trips to the
  resolver.
- You need a high-resolution interactive viewer with full MS/MS-specific annotation
  — existing interactive viewers (PDV, IPSA, Lorikeet) target that use case rather
  than unified retrieval.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves metabolomics-extended USIs against GNPS, MassBank,
    MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench data sources and returns
    spectra via /json/, /csv/, /proxi/v0.1/spectra, /png/, /svg/, /qrcode/, /spectrum/
    and /mirror/ URL endpoints
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: Universal Spectrum Identifier standard (HUPO-PSI), extended for metabolomics
    use cases, used as the query key for retrieval
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:5fd76310a245d56703a7af16f5393616e58bc1645267d0cedf647807ecf77448
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
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolomics-repository-data-access

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve tandem MS spectra held in metabolomics data repositories (GNPS, MassIVE, MetaboLights, Metabolomics Workbench, MassBank, MS2LDA/MOTIFDB) by constructing metabolomics-extended USI identifiers and querying the Metabolomics Spectrum Resolver web service endpoints, yielding machine- and human-readable spectra in a unified fashion.

## When to use

Use when a spectrum of interest lives in a metabolomics repository (e.g. a MassIVE/GNPS dataset accession with a compound name and scan number, a GNPS library accession, a MetaboLights or Metabolomics Workbench dataset spectrum) and you need programmatic, unified retrieval via a URL rather than navigating each repository's native interface.

## When NOT to use

- You need unambiguous, database-independent spectrum identification via SPLASH — SPLASH requires a complex hashing operation and cannot be trivially produced, unlike USIs.
- The spectrum is already in hand (local mzML/MGF peak list) and only needs rendering/annotation customization — direct spectrum_utils plotting avoids network round-trips to the resolver.
- You need a high-resolution interactive viewer with full MS/MS-specific annotation — existing interactive viewers (PDV, IPSA, Lorikeet) target that use case rather than unified retrieval.

## Inputs

- Metabolomics USI identifier string, e.g. mzspec:MSV000082791:(-)-epigallocatechin:scan:2 (MetaboLights/MassIVE dataset spectrum)
- GNPS library accession USI, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077
- MassBank accession USI, e.g. mzspec:MASSBANK::accession:SM858102
- Metabolomics Workbench dataset USI, e.g. mzspec:MSV000082680:iPSC-T1R1:scan:3
- MS2LDA/MOTIFDB motif USI, e.g. mzspec:MS2LDA:TASK-190:accession:43062
- ProteomeXchange dataset USI, e.g. mzspec:PXD000561:Adult_Frontalcortex_bRP_Elite_85_f09:scan:17555

## Outputs

- Machine-readable spectrum JSON containing m/z and intensity peak pairs (from /json/ endpoint)
- CSV-formatted peak list (from /csv/ endpoint)
- PROXI-compliant spectra JSON (from /proxi/v0.1/spectra endpoint)
- Rendered PNG or SVG spectrum image (from /png/ and /svg/ endpoints)
- QR code encoding the USI (from /qrcode/ endpoint)

## How to apply

Build a metabolomics USI of the form mzspec:<repo>:<filename-or-accession>:scan:<scan number> (or accession:<library accession> for GNPS/MassBank/MOTIFDB entries) — e.g. mzspec:MSV000082791:(-)-epigallocatechin:scan:2 for a MetaboLights dataset spectrum or mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 for a GNPS library spectrum — following the MassIVE/GNPS identifier format extended for metabolomics use cases. Issue an HTTP GET to the appropriate Metabolomics Spectrum Resolver endpoint (https://metabolomics-usi.ucsd.edu/) with the USI as query parameter: /json/ or the PROXI endpoint /proxi/v0.1/spectra for machine-readable spectra, /csv/ for tabular peaks, /png/ or /svg/ for rendered figures. Parse the returned JSON to extract the m/z–intensity peak list and write it to file. Note the README caveat that identifiers are draft USIs and may be specified as 'mzdraft' rather than 'mzspec' since they are subject to change.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves metabolomics-extended USIs against GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench data sources and returns spectra via /json/, /csv/, /proxi/v0.1/spectra, /png/, /svg/, /qrcode/, /spectrum/ and /mirror/ URL endpoints) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (Universal Spectrum Identifier standard (HUPO-PSI), extended for metabolomics use cases, used as the query key for retrieval)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:MSV000082791:(-)-epigallocatechin:scan:2' -o MSV000082791_epigallocatechin_scan2_peaks.json
```

## Evaluation signals

- The /json/ endpoint returns a parseable JSON object whose peak list is non-empty and contains matching m/z and intensity pair counts
- The constructed USI string follows the documented repository-specific pattern (e.g. mzspec:<MSV identifier>:<Filename>:scan:<Scan Number> for MetaboLights/Metabolomics Workbench)
- The /csv/ output row count equals the peak count reported in the /json/ peak list
- Rendering the same USI through /png/ and /json/ yields a consistent peak set (same m/z values present in both representations)
- HTTP response code 200 for the resolver GET request; a 4xx response typically indicates a malformed USI or unsupported repository block

## Limitations

- Identifiers are based on draft USI and draft Metabolomics USI identifiers and are subject to change — they may need to be specified as 'mzdraft' instead of 'mzspec' in the first block.
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support metabolomics repositories; the metabolomics extension is a workaround.
- SPLASH identifiers, while unambiguous, cannot be trivially produced because generation involves a complex hashing operation.

## Evidence

- [intro] USI currently mainly supports proteomics data deposited via ProteomeXchange.: [evidence span withheld — anchor status: not_found]
