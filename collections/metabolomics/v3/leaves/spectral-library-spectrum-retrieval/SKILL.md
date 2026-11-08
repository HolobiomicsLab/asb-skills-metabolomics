---
name: spectral-library-spectrum-retrieval
description: Use when you need to fetch and visualize a specific tandem MS/MS spectrum
  addressed by a USI (e.g. a GNPS library accession CCMSLIB... or a MetaboLights MSV
  dataset spectrum), or compare an experimental spectrum against its library match
  as a mirror plot.
when_to_use_negative:
- You need unambiguous, database-independent spectrum identifiers generated programmatically
  — SPLASH hashing, not easy-to-construct USIs, is the alternative the paper contrasts
- You need highly customized publication figures via generic vector editors (Adobe
  Illustrator, Inkscape) rather than the Resolver's parameterized rendering
- Input spectra are not in one of the supported sources (GNPS, MassBank, MS2LDA/MOTIFDB,
  MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves metabolomics-extended USIs, retrieves spectra from
    repositories, and renders single and mirror plots with cosine similarity
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library used for the plotting backend and dot-product (cosine) spectral
    matching in the mirror plot
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics,
    used to address spectra across resources
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:2cecb86a9c7222a28f35f4df7dfc7692e3e55efc6ec8baa3829f30fd5f01211d
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spectral-library-spectrum-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve MS/MS spectra from metabolomics repositories (GNPS libraries, MassIVE, MetaboLights, MassBank, MS2LDA, Metabolomics Workbench) via metabolomics-extended Universal Spectrum Identifiers and render them, including mirror plots with cosine similarity against a reference spectrum, using the Metabolomics Spectrum Resolver web service.

## When to use

Use when you need to fetch and visualize a specific tandem MS/MS spectrum addressed by a USI (e.g. a GNPS library accession CCMSLIB... or a MetaboLights MSV dataset spectrum), or compare an experimental spectrum against its library match as a mirror plot.

## When NOT to use

- You need unambiguous, database-independent spectrum identifiers generated programmatically — SPLASH hashing, not easy-to-construct USIs, is the alternative the paper contrasts
- You need highly customized publication figures via generic vector editors (Adobe Illustrator, Inkscape) rather than the Resolver's parameterized rendering
- Input spectra are not in one of the supported sources (GNPS, MassBank, MS2LDA/MOTIFDB, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)

## Inputs

- Metabolomics-extended USI strings, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:<accession>, mzspec:MASSBANK::accession:<accession>, mzspec:<MSV identifier>:<filename>:scan:<scan number>
- Two USIs (usi1, usi2) for mirror plot comparison of an experimental spectrum against a spectral library match

## Outputs

- Rendered single-spectrum figures as PNG or SVG
- Mirror plot (SVG/PNG) with cosine similarity and matching peaks highlighted, e.g. mirror_match.png
- Machine-readable spectrum data via /json/, /csv/, or /proxi/v0.1/spectra endpoints
- QR code embedding via /qrcode/

## How to apply

Construct a metabolomics-extended USI for each spectrum, e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 for a GNPS library entry or mzspec:MSV000082791:(-)-epigallocatechin:scan:2 for a MetaboLights dataset spectrum. Send HTTP GET requests to the Spectrum Resolver endpoints (/spectrum/, /json/, /csv/, /svg/, /png/) with the usi query parameter, or to /svg/mirror and /png/mirror with usi1 and usi2 parameters to render two spectra as a mirror plot. The mirror plot computes the standard cosine similarity (spectrum_utils dot-product spectral matching) between the two peak lists and highlights matching peaks so shared fragments are visually paired between the top and bottom spectra. Tune rendering with query parameters such as mz_min, mz_max, max_intensity, grid, annotate_peaks, annotate_threshold, annotation_rotation, and plot_title. For machine-readable retrieval use the /json/ or /proxi/v0.1/spectra endpoints; save the rendered figure (e.g. mirror_match.png) and verify it shows both spectra with cosine similarity and highlighted matching peaks.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves metabolomics-extended USIs, retrieves spectra from repositories, and renders single and mirror plots with cosine similarity) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library used for the plotting backend and dot-product (cosine) spectral matching in the mirror plot)
- **USI** (HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics, used to address spectra across resources)

## Examples

```
https://metabolomics-usi.ucsd.edu/svg/mirror?usi1=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077&usi2=mzspec:MSV000082791:(-)-epigallocatechin:scan:2
```

## Evaluation signals

- The mirror plot figure file (e.g. mirror_match.png) exists and contains both spectra rendered as top and bottom halves
- The reported cosine similarity is present and matching peaks are visibly highlighted/paired between the two spectra
- The USI resolves without error from the intended source (GNPS library, MassIVE, MetaboLights, MassBank, MS2LDA, Metabolomics Workbench)
- Requested rendering parameters (mz_min/mz_max, max_intensity, grid, annotations) are reflected in the returned SVG/PNG
- Expert review of the rendered image confirms the spectra match the expected accession/scan

## Limitations

- The identifiers are based on draft USI and draft Metabolomics USI identifiers, subject to change; some identifiers may need the mzdraft prefix instead of mzspec in the first block
- The native USI standard mainly supports proteomics spectral data via ProteomeXchange and does not natively support metabolomics repositories, hence the metabolomics extension
- Cosine similarity in the mirror plot is the standard dot-product match; match quality assessment depends on visually inspecting highlighted peak pairs
- No changelog or stated software version in the repository discussion

## Evidence

- [other] The Spectrum Resolver's mirror plot reports the similarity between the two spectra based on the standard cosine similarity and highlights the matching peaks between the two spectra to assess the match quality.: [evidence span withheld — anchor status: not_found]
- [intro] spectrum_utils plotting backend: [evidence span withheld — anchor status: not_found]
