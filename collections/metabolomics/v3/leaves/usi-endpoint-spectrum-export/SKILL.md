---
name: usi-endpoint-spectrum-export
description: Use when a spectrum is addressable by a USI (GNPS, MassBank, MassIVE,
  MetaboLights, Metabolomics Workbench, MS2LDA, ProteomeXchange) and a customized
  figure or exportable peak list (SVG/PNG/CSV/JSON) is needed for publication.
when_to_use_negative:
- The spectrum is only available in a local, non-deposited file format with no USI-resolvable
  source — the Resolver retrieves spectra from supported repositories (GNPS, MassBank,
  MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench) rather than from arbitrary
  local files
- Generic figure post-processing is needed beyond spectral rendering — generic vector
  editors like Adobe Illustrator or Inkscape lack MS/MS specific features and the
  Resolver is intended to avoid that time investment, not replace full graphics editing
- High-throughput interactive data exploration of a raw LC-MS/MS run is the goal rather
  than generating a single resolved spectrum figure
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose /svg/, /png/, /csv/, /json/, /mirror/, and /qrcode/ endpoints
    render and export USI-resolved MS/MS spectra
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library implementing the underlying spectrum plotting functionality
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:d69ff75ea6d575d02564ba5855bcf8e29db75b9ebf623055113da692a274d264
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
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# usi-endpoint-spectrum-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render publication-ready MS/MS spectra and machine-readable peak lists from a Universal Spectrum Identifier (USI) via the Metabolomics Spectrum Resolver web service's /svg/, /png/, /csv/, and /json/ endpoints, with tunable plotting parameters and optional mirror plots.

## When to use

Use when a spectrum is addressable by a USI (GNPS, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA, ProteomeXchange) and a customized figure or exportable peak list (SVG/PNG/CSV/JSON) is needed for publication.

## When NOT to use

- The spectrum is only available in a local, non-deposited file format with no USI-resolvable source — the Resolver retrieves spectra from supported repositories (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench) rather than from arbitrary local files
- Generic figure post-processing is needed beyond spectral rendering — generic vector editors like Adobe Illustrator or Inkscape lack MS/MS specific features and the Resolver is intended to avoid that time investment, not replace full graphics editing
- High-throughput interactive data exploration of a raw LC-MS/MS run is the goal rather than generating a single resolved spectrum figure

## Inputs

- USI string referencing a spectrum (GNPS task/library, MassBank, MS2LDA/MOTIFDB, MassIVE, MetaboLights, Metabolomics Workbench, or ProteomeXchange)
- Peak list from a prior processing step, cast into a USI (e.g. a GNPS TASK scan identifier)
- Optional plotting parameters: mz_min, mz_max, max_intensity, width, height, grid, plot_title, annotate_peaks, annotate_precision, annotate_threshold, annotation_rotation

## Outputs

- SVG vector spectrum figure (or mirror plot) suitable for publication
- PNG raster spectrum figure
- CSV comma-separated peak list
- JSON machine-readable peak list
- Optional QR code embedding linking to an interactive spectrum view

## How to apply

Construct a USI referencing the target spectrum (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 or mzspec:MASSBANK::accession:SM858102), noting that metabolomics identifiers currently use the 'mzdraft' prefix because the metabolomics USI extension is a draft. Render the spectrum through the /svg/ or /png/ endpoints, customizing with query parameters such as mz_min, mz_max, max_intensity, width, height, grid, plot_title, annotate_peaks (a nested list of m/z values), annotate_precision, annotate_threshold, and annotation_rotation. For library-vs-experimental comparisons, use /svg/mirror or /png/mirror with usi1 and usi2, optionally passing two nested lists to annotate_peaks for top/bottom spectra. Export raw peaks as machine-readable data via the /json/ and /csv/ endpoints (a /proxi/v0.1/spectra endpoint is also available), and optionally generate a QR code via /qrcode/ so published figures link back to an interactive view. Plotting is implemented in Python using spectrum_utils, Matplotlib, and Seaborn.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose /svg/, /png/, /csv/, /json/, /mirror/, and /qrcode/ endpoints render and export USI-resolved MS/MS spectra) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library implementing the underlying spectrum plotting functionality)

## Examples

```
https://metabolomics-usi.ucsd.edu/svg/?usi=mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943&mz_min=550&mz_max=800&width=4&height=4
```

## Evaluation signals

- All four output files exist: the SVG, PNG, CSV, and JSON exports each return successfully for the same USI
- The rendered figure reflects requested parameters — e.g. the m/z axis starts at mz_min=550 and ends at mz_max=800, or the y-axis is clipped at max_intensity=50 when intensity zoom is applied
- Custom annotate_peaks labels appear only at the specified m/z values (e.g. 463.297, 708.463, 816.474, 1042.5699) with annotate_precision decimal places
- CSV/JSON peak m/z and intensity values match the peaks drawn in the SVG/PNG figure for the same USI
- Mirror plot output shows the two USI spectra (usi1 top, usi2 bottom) with shared m/z axis

## Limitations

- Metabolomics USI identifiers are draft ('mzdraft' instead of 'mzspec') and subject to change
- The USI standard mainly supports proteomics data via ProteomeXchange and does not natively support metabolomics repositories without the metabolomics extension
- No software version is stated for the Metabolomics Spectrum Resolver; the service is only available as a freely available open-source web service

## Evidence

- [intro] Plotting functionality is implemented in Python using spectrum_utils, Matplotlib, and Seaborn.: [evidence span withheld — anchor status: not_found]
