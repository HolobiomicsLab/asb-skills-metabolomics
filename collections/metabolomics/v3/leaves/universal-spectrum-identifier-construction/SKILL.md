---
name: universal-spectrum-identifier-construction
description: Use when a specific tandem MS spectrum (e.g., a GNPS library entry CCMSLIB
  accessions, a MassIVE/MetaboLights MSV dataset scan, a MassBank accession, or an
  MS2LDA motif) must be addressed by URL for retrieval or visualization.
when_to_use_negative:
- When a database-independent, content-hashed identifier is required — SPLASH identifiers
  exist for that purpose, but 'generating a SPLASH identifier includes a complex hashing
  operation', so they are not trivially produced.
- When the spectrum is not held in a USI-supported source (GNPS, MassBank, MassIVE,
  MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB, ProteomeXchange) or lacks
  a resolvable scan/accession locator.
- When stable, finalized identifiers are required — the README warns these metabolomics
  identifiers are draft ('mzdraft') and subject to change.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that accepts the constructed USI as a query parameter and retrieves,
    resolves, and displays the corresponding tandem MS spectrum from the underlying
    repository
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: The identifier specification being applied; extended by this work for metabolomics
    use cases beyond its original ProteomeXchange proteomics scope
- name: spectrum_utils
  role: Python library implementing the plotting and dot-product spectral matching
    underneath the resolver's rendered figures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:412cb1f781b9f24d63fd9934b962fadeb6c20890a34ca2b26dced6e647fbe50a
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
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# universal-spectrum-identifier-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct USI-formatted identifiers (mzspec:...) to address individual MS/MS spectra held in metabolomics and proteomics repositories — GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA, MOTIFDB, ProteomeXchange — so they can be retrieved and rendered by the Metabolomics Spectrum Resolver web service. USIs are easy to build by hand, unlike SPLASH hashes, which require a complex hashing operation.

## When to use

Use when a specific tandem MS spectrum (e.g., a GNPS library entry CCMSLIB accessions, a MassIVE/MetaboLights MSV dataset scan, a MassBank accession, or an MS2LDA motif) must be addressed by URL for retrieval or visualization.

## When NOT to use

- When a database-independent, content-hashed identifier is required — SPLASH identifiers exist for that purpose, but 'generating a SPLASH identifier includes a complex hashing operation', so they are not trivially produced.
- When the spectrum is not held in a USI-supported source (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB, ProteomeXchange) or lacks a resolvable scan/accession locator.
- When stable, finalized identifiers are required — the README warns these metabolomics identifiers are draft ('mzdraft') and subject to change.

## Inputs

- GNPS spectral library accession (e.g., CCMSLIB00005436077)
- MassBank accession (e.g., SM858102, BSU00002)
- MassIVE/GNPS MSV identifier plus filename and scan number (e.g., MSV000079514, MSV000082791)
- MetaboLights dataset identifier, filename, and scan number (e.g., MSV000082791:(-)-epigallocatechin:scan:2)
- Metabolomics Workbench MSV identifier, filename, and scan number (e.g., MSV000082680:iPSC-T1R1:scan:3)
- ProteomeXchange PXD identifier, filename, and scan number (e.g., PXD000561)
- GNPS/MassIVE task ID, file name, and scan number
- MS2LDA TASK or MOTIFDB accession

## Outputs

- USI string in mzspec:... syntax (e.g., mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077)
- URL-encoded resolver request targeting /spectrum/, /png/, /svg/, /json/, /csv/, /qrcode/, or /mirror/ endpoints

## How to apply

Build the identifier in the mzspec:<source>:<dataset/locator>:<type>:<accession> syntax, choosing the pattern that matches the spectrum's source. Use mzspec:GNPS:GNPS-LIBRARY:accession:<CCMSLIB ID> for GNPS library spectra; mzspec:MASSBANK::accession:<MassBank ID> for MassBank; mzspec:<MSV identifier>:<Filename>:scan:<scan number> for MetaboLights and Metabolomics Workbench dataset spectra; mzspec:PXDxxxxxx:<file>:scan:<n> or mzspec:MSVxxxxxx:<file>:scan:<n> for ProteomeXchange/MassIVE repository data; and mzspec:GNPS:TASK-<task ID>-<file name>:scan:<n> for GNPS/MassIVE task output. Because these metabolomics identifiers are based on a draft extension of the PSI-USI standard, the README notes they may be specified as mzdraft instead of mzspec and are subject to change. Pass the finished USI as a usi (or usi1/usi2 for mirror plots) query parameter to the resolver's /spectrum/, /png/, /svg/, /json/, /csv/, /qrcode/, or /mirror/ endpoints. Optionally append plotting parameters such as mz_min, mz_max, max_intensity, annotate_peaks, or plot_title to control rendering.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that accepts the constructed USI as a query parameter and retrieves, resolves, and displays the corresponding tandem MS spectrum from the underlying repository) — github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (The identifier specification being applied; extended by this work for metabolomics use cases beyond its original ProteomeXchange proteomics scope)
- **spectrum_utils** (Python library implementing the plotting and dot-product spectral matching underneath the resolver's rendered figures)

## Examples

```
https://metabolomics-usi.ucsd.edu/svg/mirror?usi1=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077&usi2=mzspec:MSV000082791:(-)-epigallocatechin:scan:2
```

## Evaluation signals

- The constructed USI parses into the documented field pattern for its source type (e.g., mzspec:<MetaboLights MSV identifier>:<Filename>:scan:<Scan Number>)
- A request to https://metabolomics-usi.ucsd.edu/spectrum/?usi=<USI> returns a rendered spectrum rather than a resolution error
- For mirror-plot use, /svg/mirror or /png/mirror with usi1 and usi2 resolves both spectra and reports a standard cosine similarity with matching peaks highlighted
- The USI's first block matches mzspec (or mzdraft for draft metabolomics identifiers per the README warning)
- The /json/ or /proxi/v0.1/spectra endpoint returns machine-readable peak data for the addressed spectrum

## Limitations

- These identifiers are based on draft USI and draft Metabolomics USI identifiers and 'will be specified as mzdraft instead of mzspec in the first block' pending standardization, so they are subject to change.
- The native PSI-USI standard 'currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support metabolomics repositories'; metabolomics support here comes from this tool's extension.
- The GNPS library pattern ignores the library-name field (mzspec:GNPS:<library name (ignored)>:accession:<CCMSLIB ID>).
- MassIVE/GNPS repository spectra must follow the proteomics USI standard for dataset/file/scan addressing.

## Evidence

- [intro] USI origin and rationale: [evidence span withheld — anchor status: not_found]
- [intro] USI proteomics-only limitation: [evidence span withheld — anchor status: not_found]
