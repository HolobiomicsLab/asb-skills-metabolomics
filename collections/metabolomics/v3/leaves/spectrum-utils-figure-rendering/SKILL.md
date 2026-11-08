---
name: spectrum-utils-figure-rendering
description: Use when you need to visualize tandem MS spectra addressed by USIs from
  GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA, or ProteomeXchange,
  or produce a mirror plot quantifying similarity between two spectra.
when_to_use_negative:
- You need interactive exploration or MS/MS-specific editing of figures beyond the
  resolver's parameters — generic editors like Adobe Illustrator or Inkscape or interactive
  viewers (PDV, IPSA, Lorikeet) may fit, though they lack MS/MS-specific features
- You must generate SPLASH (SPectraL hASH) identifiers — the resolver addresses spectra
  by USI, and SPLASH generation involves a complex hashing operation not provided
  here
- The spectrum is not addressable by one of the supported USI types (GNPS tasks/libraries,
  MassBank, ProteomeXchange, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB)
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves USIs to spectra from multiple repositories and renders
    single-spectrum and mirror plots via /png/, /svg/, /json/, /csv/, /qrcode/, /proxi/v0.1/spectra,
    /spectrum/ and /mirror/ endpoints
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python plotting library implementing the spectral rendering and dot-product
    cosine similarity used by the resolver's mirror-match figure
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics,
    used to address each spectrum as a query parameter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:84b9ffc9249daf168a0fc024d18c011346aac356249fd1610b9fc2e4eae5ae1e
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectrum-utils-figure-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render publication-ready MS/MS spectra and mirror plots via the Metabolomics Spectrum Resolver's USI-addressed endpoints (/png/, /svg/, /mirror/), using spectrum_utils/Matplotlib/Seaborn plotting under the hood, with cosine-similarity mirror matches to compare experimental spectra against spectral library entries.

## When to use

Use when you need to visualize tandem MS spectra addressed by USIs from GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA, or ProteomeXchange, or produce a mirror plot quantifying similarity between two spectra.

## When NOT to use

- You need interactive exploration or MS/MS-specific editing of figures beyond the resolver's parameters — generic editors like Adobe Illustrator or Inkscape or interactive viewers (PDV, IPSA, Lorikeet) may fit, though they lack MS/MS-specific features
- You must generate SPLASH (SPectraL hASH) identifiers — the resolver addresses spectra by USI, and SPLASH generation involves a complex hashing operation not provided here
- The spectrum is not addressable by one of the supported USI types (GNPS tasks/libraries, MassBank, ProteomeXchange, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB)

## Inputs

- Metabolomics-extended USI strings (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:..., mzspec:MSV000082791:<filename>:scan:<n>, mzspec:MASSBANK::accession:..., mzspec:MOTIFDB::accession:...)
- Rendering parameter settings (mz_min, mz_max, max_intensity, grid, annotate_peaks, annotate_precision, annotation_rotation, annotate_threshold, plot_title, width, height)
- Two USIs (usi1, usi2) for mirror-plot similarity comparison

## Outputs

- PNG or SVG spectrum figures (publication-ready, vector graphics for high resolution)
- Mirror plot image (e.g. mirror_match.png) with cosine similarity and highlighted matching peaks
- Machine-readable spectrum data via /json/, /csv/, or /proxi/v0.1/spectra endpoints
- QR code images via /qrcode/

## How to apply

Construct a metabolomics-extended USI for each spectrum, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 for a GNPS library spectrum or mzspec:MSV000082791:(-)-epigallocatechin:scan:2 for a MetaboLights dataset spectrum. Issue HTTP requests to the Metabolomics Spectrum Resolver endpoints (/png/, /svg/, /json/, /csv/, /qrcode/, /proxi/v0.1/spectra, /spectrum/) passing the USI as the 'usi' query parameter, or to /svg/mirror and /png/mirror with 'usi1' and 'usi2' parameters for a mirror match. Tune rendering with query parameters: mz_min/mz_max for mass range filtering, max_intensity for intensity zoom, grid=false, annotate_peaks as nested lists of m/z values, annotate_precision, annotation_rotation, annotate_threshold, and plot_title. For mirror plots, the resolver computes the standard cosine similarity (spectrum_utils dot-product spectral matching) between the two peak lists and highlights matching peaks shared between top and bottom spectra. Save the returned image (e.g. mirror_match.png) and verify it contains both spectra with the cosine similarity and highlighted peak matches.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves USIs to spectra from multiple repositories and renders single-spectrum and mirror plots via /png/, /svg/, /json/, /csv/, /qrcode/, /proxi/v0.1/spectra, /spectrum/ and /mirror/ endpoints) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python plotting library implementing the spectral rendering and dot-product cosine similarity used by the resolver's mirror-match figure)
- **USI** (HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics, used to address each spectrum as a query parameter)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/png/mirror?usi1=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077&usi2=mzspec:MSV000082791:(-)-epigallocatechin:scan:2' -o mirror_match.png
```

## Evaluation signals

- The output figure file (e.g. mirror_match.png) exists and renders both spectra as a mirror plot with the cosine similarity shown
- Matching peaks are highlighted so shared fragments are visually paired between the top and bottom spectra
- Vector output (SVG) is produced when high-resolution publication figures are required
- Rendering parameters take effect, e.g. mz_min/mz_max restrict the mass range, max_intensity zooms intensity, and custom annotate_peaks m/z values appear as annotations
- Each USI resolves correctly for its source (GNPS library accession, MassIVE/MetaboLights MSV identifier with filename and scan number) per the metabolomics-extended USI formats documented in the README

## Limitations

- Metabolomics USIs are based on draft USI and draft Metabolomics USI identifiers, subject to change; some are specified as 'mzdraft' instead of 'mzspec'
- The USI standard natively supports proteomics data via ProteomeXchange and requires metabolomics-specific extensions for metabolomics repositories
- No software version stated for the Metabolomics Spectrum Resolver; no changelog provided
- The USI format requires the correct per-source syntax (e.g. GNPS library name is ignored; MetaboLights needs MSV identifier, filename, and scan number)

## Evidence

- [intro] Plotting functionality is implemented in Python using spectrum_utils, Matplotlib, and Seaborn.: [evidence span withheld — anchor status: not_found]
- [methods] resolver endpoints: [evidence span withheld — anchor status: not_found]
