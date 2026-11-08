---
name: spectrum-visualization-parameter-tuning
description: Use when a tandem MS spectrum must be turned into a publication-ready
  figure with controlled m/z range, intensity zoom, sizing, grid, and peak-annotation
  settings, and exported in vector/raster/machine-readable formats.
when_to_use_negative:
- The spectrum viewer needs interactive exploration of raw vendor files rather than
  static publication figures — the article notes existing MS/MS viewers such as PDV,
  IPSA, and Lorikeet target interactive visualization.
- A custom MS/MS figure is needed for spectra not resolvable through supported sources
  (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
  — no USI type matches the data.
- The task is general figure editing rather than MS/MS-specific rendering; the article
  warns generic vector editors like Adobe Illustrator or Inkscape lack MS/MS-specific
  features, and this skill does not substitute for post-hoc vector editing.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose /svg/, /png/, /svg/mirror, /png/mirror, /csv/, and /json/
    endpoints render the customized spectrum figures and export the peak data
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python plotting library implementing the underlying spectrum rendering for
    the Resolver's visualization endpoints
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:4129344e5ae8a57c72fe29eb3aee2db1f3bd0c2d2239fbad29a839cb04bda1a0
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectrum-visualization-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render customizable, publication-ready MS/MS spectrum figures via the Metabolomics Spectrum Resolver by tuning URL plotting parameters (mz range, intensity, size, annotations) on USI-referenced spectra, and export them as SVG/PNG images plus CSV/JSON peak data.

## When to use

Use when a tandem MS spectrum must be turned into a publication-ready figure with controlled m/z range, intensity zoom, sizing, grid, and peak-annotation settings, and exported in vector/raster/machine-readable formats.

## When NOT to use

- The spectrum viewer needs interactive exploration of raw vendor files rather than static publication figures — the article notes existing MS/MS viewers such as PDV, IPSA, and Lorikeet target interactive visualization.
- A custom MS/MS figure is needed for spectra not resolvable through supported sources (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange) — no USI type matches the data.
- The task is general figure editing rather than MS/MS-specific rendering; the article warns generic vector editors like Adobe Illustrator or Inkscape lack MS/MS-specific features, and this skill does not substitute for post-hoc vector editing.

## Inputs

- USI-referenced MS/MS spectrum (e.g. mzspec:GNPS:TASK-<id>-specs_ms.mgf:scan:1943, mzspec:MASSBANK::accession:SM858102, MetaboLights/Metabolomics Workbench MSV scans)
- Peak list file (e.g. MGF peak list from a prior task) used to construct the USI
- Visualization customization parameters (mz_min, mz_max, max_intensity, width, height, annotate_peaks, annotation_rotation, annotate_threshold, annotate_precision, grid, plot_title)

## Outputs

- SVG vector spectrum figure (single or mirror plot)
- PNG raster spectrum figure
- CSV peak list (comma-separated m/z–intensity values)
- JSON machine-readable peak list

## How to apply

Construct a USI referencing the target spectrum (e.g. mzspec:GNPS:TASK-<task id>-<file>:scan:<scan>, MassBank accession, or MetaboLights/Metabolomics Workbench MSV identifiers; note the README specifies the draft prefix 'mzdraft' instead of 'mzspec'). Render the spectrum through the /svg/ or /png/ endpoints, adding query parameters to customize the figure: mz_min and mz_max to restrict the mass range, max_intensity to zoom intensity, width and height to control figure dimensions, grid=false to remove gridlines, annotate_peaks=[[m1, m2]] to select annotated peaks, annotation_rotation (e.g. 45) to rotate labels, annotate_precision to reduce decimal places, annotate_threshold=0 to lower the label minimum intensity, and plot_title for a custom title. For comparisons of two spectra (e.g. library vs. experimental), use /svg/mirror or /png/mirror with usi1 and usi2, optionally scaling with max_intensity. Finally, export the underlying peak list through the /json/ and /csv/ endpoints so the figure and its data remain consistent. Validate by checking that all four outputs (SVG, PNG, CSV, JSON) exist and that the rendered figure reflects the requested mz range, annotation, and sizing settings.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose /svg/, /png/, /svg/mirror, /png/mirror, /csv/, and /json/ endpoints render the customized spectrum figures and export the peak data) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python plotting library implementing the underlying spectrum rendering for the Resolver's visualization endpoints)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/svg/?usi=mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943&mz_min=550&mz_max=800&max_intensity=50&annotation_rotation=45&width=4&height=4'
```

## Evaluation signals

- All four export artifacts exist after the run: SVG, PNG, CSV, and JSON for the same USI.
- Rendered figure x-axis respects the requested mz_min/mz_max window and y-axis respects max_intensity zoom.
- Only the peaks listed in annotate_peaks (or all peaks when annotate_threshold=0) carry labels, with label rotation matching annotation_rotation and decimal places matching annotate_precision.
- CSV and JSON peak lists agree with the annotated peaks in the rendered figure (same m/z–intensity values).
- Mirror plot correctly places usi1 on top and usi2 on bottom with shared m/z axis and honors intensity scaling.

## Limitations

- USI identifiers are based on draft metabolomics-USI specifications and must use the 'mzdraft' prefix instead of 'mzspec'; they are subject to change.
- The USI standard currently mainly supports proteomics spectral data via ProteomeXchange and does not natively support metabolomics repositories, hence the metabolomics extension.
- Missing metadata in the deployed service: no changelog and no stated software version; implemented as a freely available open-source web service.

## Evidence

- [intro] Plotting functionality is implemented in Python using spectrum_utils, Matplotlib, and Seaborn.: [evidence span withheld — anchor status: not_found]
