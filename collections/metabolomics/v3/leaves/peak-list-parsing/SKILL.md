---
name: peak-list-parsing
description: Use when you need the underlying peak list (m/z and intensity pairs)
  of a specific MS/MS spectrum held in GNPS, MassIVE, MassBank, MS2LDA, MetaboLights,
  or Metabolomics Workbench, identified by a metabolomics USI such as mzspec:MSV000082791:(-)-epigallocatechin:scan:2.
when_to_use_negative:
- The input is already a parsed peak list or feature table — no spectrum retrieval
  is needed.
- The spectrum resides in a repository not supported by the Resolver (USI standard
  natively supports mainly proteomics data deposited via ProteomeXchange; other sources
  require the draft metabolomics USI extension).
- You need an interactive visual inspection rather than machine-readable peaks — the
  /spectrum/ or /svg/ rendering endpoints are more appropriate.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves the metabolomics USI and returns the spectrum peak
    list as JSON via its /json/ endpoint
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:aece442e6a17f12930b1cc58febbed2ffa6e79f9271749f3fa339890fd0fd7b1
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# peak-list-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves a raw MS/MS spectrum as a machine-readable peak list (m/z–intensity pairs) from metabolomics repositories via the Metabolomics Spectrum Resolver /json/ endpoint, converting a metabolomics USI into structured peak data.

## When to use

Use when you need the underlying peak list (m/z and intensity pairs) of a specific MS/MS spectrum held in GNPS, MassIVE, MassBank, MS2LDA, MetaboLights, or Metabolomics Workbench, identified by a metabolomics USI such as mzspec:MSV000082791:(-)-epigallocatechin:scan:2.

## When NOT to use

- The input is already a parsed peak list or feature table — no spectrum retrieval is needed.
- The spectrum resides in a repository not supported by the Resolver (USI standard natively supports mainly proteomics data deposited via ProteomeXchange; other sources require the draft metabolomics USI extension).
- You need an interactive visual inspection rather than machine-readable peaks — the /spectrum/ or /svg/ rendering endpoints are more appropriate.

## Inputs

- Metabolomics USI string (e.g. 'mzspec:MSV000082791:(-)-epigallocatechin:scan:2') following MassIVE/GNPS, MetaboLights, Metabolomics Workbench, GNPS library, MassBank, MS2LDA, or MOTIFDB identifier formats
- MassIVE/GNPS dataset identifier (e.g. MSV000082791) with filename and scan number

## Outputs

- JSON file containing the parsed peak list of m/z and intensity pairs (e.g. MSV000082791_epigallocatechin_scan2_peaks.json)
- Optionally CSV peak lists via the /csv/ endpoint

## How to apply

Construct a metabolomics USI in the repository-specific format (e.g. mzspec:<MetaboLights MSV identifier>:<Filename>:scan:<Scan Number> for MetaboLights/Metabolomics Workbench data, or the MassIVE/GNPS repository format such as mzspec:MSV000082791:(-)-epigallocatechin:scan:2). Issue an HTTP GET request to the Spectrum Resolver's /json/ URL endpoint (https://metabolomics-usi.ucsd.edu/json/) with the USI as the 'usi' query parameter, so the service resolves the identifier across the supported repositories and returns the spectrum in machine-readable JSON. Parse the JSON response to extract the peak list of m/z and intensity pairs, then save it (e.g. as MSV000082791_epigallocatechin_scan2_peaks.json). If structured retrieval through the standard interface is preferred, the /proxi/v0.1/spectra endpoint is also available among the service's URL endpoints (/png/, /svg/, /json/, /proxi/v0.1/spectra, /csv/, /qrcode/, /spectrum/, /mirror/). Rationale: the USI standard was extended for metabolomics because it originally only natively supported proteomics data in ProteomeXchange, and the Resolver provides consistent human- and machine-readable export.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves the metabolomics USI and returns the spectrum peak list as JSON via its /json/ endpoint) — https://github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:MSV000082791:(-)-epigallocatechin:scan:2' -o MSV000082791_epigallocatechin_scan2_peaks.json
```

## Evaluation signals

- The output JSON file exists and parses as valid JSON.
- The peak list is non-empty and each entry contains both an m/z value and an intensity value.
- The returned peaks correspond to the scan specified in the USI (e.g. scan 2 of MSV000082791 for (-)-epigallocatechin).
- The USI uses the correct repository format; identifiers are draft metabolomics USIs, so check the 'mzspec' vs 'mzdraft' prefix caveat.
- Equivalent retrieval via /proxi/v0.1/spectra or /csv/ endpoints yields consistent peak positions.

## Limitations

- Metabolomics USI identifiers are based on draft USI/draft Metabolomics USI identifiers and are subject to change; the README notes they may need to be specified as 'mzdraft' instead of 'mzspec'.
- The USI standard natively does not support spectral data stored in metabolomics repositories; only the Resolver's extension covers MassIVE, MetaboLights, and Metabolomics Workbench sources.
- No software version is stated for the Metabolomics Spectrum Resolver; it is a freely available, open-source web service, so endpoint behavior may drift.

## Evidence

- [intro] the USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support spectral data stored in metabolomics repositories.: [evidence span withheld — anchor status: not_found]
