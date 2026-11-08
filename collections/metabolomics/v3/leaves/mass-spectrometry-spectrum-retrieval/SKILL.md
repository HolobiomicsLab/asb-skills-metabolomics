---
name: mass-spectrometry-spectrum-retrieval
description: Use when you need to programmatically fetch a specific MS/MS spectrum
  from a public metabolomics or proteomics repository and you know (or can construct)
  its USI — e.g. to feed a peak list into a downstream scoring or visualization pipeline.
when_to_use_negative:
- The spectrum identifier is a SPLASH hash rather than a USI — SPLASH requires a complex
  hashing operation and is not trivially constructed, so a different retrieval path
  is needed
- The data are stored only as local raw files not deposited in a supported repository
  (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange),
  since the resolver can only retrieve deposited spectra
- You need MS/MS-specific interactive figure customization beyond the resolver's plotting
  parameters — generic vector editors like Inkscape lack MS/MS features but offer
  higher customization at significant time cost
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that accepts a metabolomics-extended USI and returns the spectrum
    as JSON, CSV, PNG, SVG, mirror plot, PROXI response, or QR code
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library used to implement the resolver's plotting functionality; downstream
    users can apply it to retrieved peak lists for further processing (e.g. shifted
    cosine similarity)
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:0364900676ae07f55668640605b42fe773b37873221c39813901f50ad3144375
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
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-spectrum-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve MS/MS spectra from heterogeneous repositories (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange) in a unified fashion by constructing a metabolomics-extended Universal Spectrum Identifier (USI) and querying the Metabolomics Spectrum Resolver web service for machine-readable (/json/, /csv/) or image (/png/, /svg/) output.

## When to use

Use when you need to programmatically fetch a specific MS/MS spectrum from a public metabolomics or proteomics repository and you know (or can construct) its USI — e.g. to feed a peak list into a downstream scoring or visualization pipeline.

## When NOT to use

- The spectrum identifier is a SPLASH hash rather than a USI — SPLASH requires a complex hashing operation and is not trivially constructed, so a different retrieval path is needed
- The data are stored only as local raw files not deposited in a supported repository (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange), since the resolver can only retrieve deposited spectra
- You need MS/MS-specific interactive figure customization beyond the resolver's plotting parameters — generic vector editors like Inkscape lack MS/MS features but offer higher customization at significant time cost

## Inputs

- Metabolomics-extended USI string (e.g. `mzspec:MSV000082791:(-)-epigallocatechin:scan:2` or draft `mzdraft:...` variant)
- USI query parameter appended to one of the resolver endpoints (/json/, /csv/, /png/, /svg/, /svg/mirror, /png/mirror, /proxi/v0.1/spectra, /qrcode/, /spectrum/)
- Optional plotting parameters: mz_min, mz_max, max_intensity, annotate_peaks, annotate_precision, annotation_rotation, annotate_threshold, grid, plot_title, width, height

## Outputs

- Machine-readable spectrum as JSON peak list (m/z and intensity pairs) from /json/
- CSV spectrum table from /csv/
- Raster (PNG) or vector (SVG) spectrum image for publication-ready figures
- Mirror plot image comparing two spectra (/svg/mirror, /png/mirror)
- PROXI v0.1 compatible spectra response
- Embeddable QR code linking to the interactive spectrum viewer

## How to apply

Construct the USI in the repository-specific format (e.g. `mzspec:<MSV identifier>:<Filename>:scan:<Scan Number>` for MetaboLights or Metabolomics Workbench data, `mzspec:PXD<id>:<file>:scan:<n>` for ProteomeXchange, `mzspec:GNPS:GNPS-LIBRARY:accession:<accession>` for GNPS library spectra) and pass it as the `usi` query parameter to the Spectrum Resolver endpoint for the desired representation: /json/ or /csv/ for machine-readable peak lists, /png/ or /svg/ for images, /svg/mirror or /png/mirror for two-spectrum mirror plots, /proxi/v0.1/spectra for PROXI-compatible output, /qrcode/ for embeddable QR codes. Plotting parameters such as `mz_min`, `mz_max`, `max_intensity`, `annotate_peaks`, `annotate_precision`, `annotation_rotation`, `annotate_threshold`, `grid`, and `plot_title` can be appended to control the rendered figure. The tool was built because the USI standard natively supports proteomics data via ProteomeXchange but not metabolomics repositories, so the metabolomics extension is required for MassIVE/GNPS/MetaboLights/Metabolomics Workbench identifiers. Note the README warning that these are draft identifiers specified as `mzdraft` in the first block pending finalization. Validate that the returned payload parses and contains the expected peak list (m/z and intensity pairs).

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that accepts a metabolomics-extended USI and returns the spectrum as JSON, CSV, PNG, SVG, mirror plot, PROXI response, or QR code) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library used to implement the resolver's plotting functionality; downstream users can apply it to retrieved peak lists for further processing (e.g. shifted cosine similarity)) — https://github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:MSV000082791:(-)-epigallocatechin:scan:2'
```

## Evaluation signals

- The returned /json/ payload parses as valid JSON and contains a non-empty peak list with m/z and intensity pairs for the requested USI
- A self-comparison via the /mirror/ endpoint (identical USIs, e.g. `mzspec:MASSBANK::accession:BSU00002` against itself) renders a mirror plot with matching peaks aligned
- Draft metabolomics identifiers use the `mzdraft` first block rather than `mzspec`, per the README warning that they are subject to change
- Endpoint-specific output matches the requested format: /svg/ returns vector graphics, /png/ returns raster, /qrcode/ returns an embeddable QR code
- USI construction follows the repository-specific format, e.g. `mzspec:<MetaboLights MSV identifier>:<Filename>:scan:<Scan Number>`

## Limitations

- Metabolomics USIs are based on draft identifiers and are subject to change; they must currently be specified as `mzdraft` instead of `mzspec` in the first block
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support metabolomics repositories, hence the extension
- SPLASH identifiers cannot be trivially produced because generation involves a complex hashing operation
- No software version, DOI, authors, or publication year are stated in the README; the README also notes no changelog is found

## Evidence

- [intro] USI standard does not natively support metabolomics repositories: [evidence span withheld — anchor status: not_found]
