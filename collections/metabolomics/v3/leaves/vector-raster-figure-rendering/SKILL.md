---
name: vector-raster-figure-rendering
description: Use when a USI-referenced tandem MS spectrum needs to be turned into
  a publication-ready figure with controlled m/z range, dimensions, peak annotations,
  or a mirror-plot comparison against a second spectrum.
when_to_use_negative:
- Input is raw vendor or mzML spectral data with no resolvable USI — the resolver
  only retrieves spectra from supported repositories and USI types
- High-throughput automated figure generation is not needed and the goal is machine-readable
  peak data only — the /csv/ or /json/ endpoints are the right choice instead of rendering
- The figure requires MS/MS-specific interactive inspection beyond static rendering
  — interactive viewers like PDV or IPSA may be more appropriate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose /svg/, /png/, /svg/mirror and /png/mirror endpoints render
    customizable spectrum figures from USI inputs
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library implementing the plotting functionality underlying the resolver's
    figure generation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:0e70d397bd4c59ef8f78a98b915f7bfacc3e0500705c9644350ed52d194fd534
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# vector-raster-figure-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render customizable, publication-ready MS/MS spectrum figures via the Metabolomics Spectrum Resolver's /svg/ and /png/ endpoints, tuning m/z range, intensity, sizing, and annotation parameters, and optionally produce mirror plots via /svg/mirror or /png/mirror.

## When to use

Use when a USI-referenced tandem MS spectrum needs to be turned into a publication-ready figure with controlled m/z range, dimensions, peak annotations, or a mirror-plot comparison against a second spectrum.

## When NOT to use

- Input is raw vendor or mzML spectral data with no resolvable USI — the resolver only retrieves spectra from supported repositories and USI types
- High-throughput automated figure generation is not needed and the goal is machine-readable peak data only — the /csv/ or /json/ endpoints are the right choice instead of rendering
- The figure requires MS/MS-specific interactive inspection beyond static rendering — interactive viewers like PDV or IPSA may be more appropriate

## Inputs

- Universal Spectrum Identifier (USI/mzdraft) string referencing a spectrum from GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, or ProteomeXchange
- Rendering parameter set (mz_min, mz_max, max_intensity, width, height, annotate_peaks, annotate_threshold, annotation_rotation, annotate_precision, plot_title, grid)
- Second USI (usi2) for mirror plots

## Outputs

- SVG vector spectrum figure
- PNG raster spectrum figure
- SVG/PNG mirror plot comparing two spectra

## How to apply

Construct a USI (mzdraft-style for metabolomics sources such as GNPS tasks, MassBank, MS2LDA, MetaboLights, or Metabolomics Workbench) referencing the target spectrum, then issue an HTTP GET to the resolver's /svg/ or /png/ endpoint. Tune rendering parameters in the URL: mz_min and mz_max to window the m/z axis (e.g. 550–800), max_intensity to zoom the y-axis, width and height for figure size, annotation_rotation (e.g. 45) and annotate_precision for label styling, annotate_peaks as a nested list of m/z values to selectively annotate, and annotate_threshold (e.g. 0) to lower the label minimum intensity. For side-by-side library-match inspection, call /svg/mirror or /png/mirror with usi1 and usi2; annotate_peaks then takes two nested lists (top and bottom spectra). Use the /svg/ output for lossless vector editing in tools like Adobe Illustrator or Inkscape, and /png/ for raster embedding; the rationale is that generic editors lack MS/MS-specific features, so customization is done at render time.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose /svg/, /png/, /svg/mirror and /png/mirror endpoints render customizable spectrum figures from USI inputs) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library implementing the plotting functionality underlying the resolver's figure generation)

## Examples

```
GET https://metabolomics-usi.ucsd.edu/svg/?usi=mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943&mz_min=550&mz_max=800&width=4&height=4&annotation_rotation=45
```

## Evaluation signals

- All requested output files (SVG and PNG) exist and are non-empty after rendering
- Rendered figure reflects requested mz_min/mz_max windowing (e.g. 550–800) and max_intensity zoom in axis ranges
- Only peaks listed in annotate_peaks carry labels; annotation_rotation and annotate_precision are visibly applied
- Mirror plot shows both usi1 (top) and usi2 (bottom) spectra with correct orientation
- SVG is vector (editable in Illustrator/Inkscape) rather than a rasterized image

## Limitations

- Metabolomics USI identifiers are draft-based and prefixed 'mzdraft' rather than 'mzspec', and are subject to change
- Generic vector graphics editors like Illustrator or Inkscape lack MS/MS-specific features, so customization must happen at render time
- Existing MS/MS viewers often miss vector graphics export, visualization customization, and high-throughput automated figure generation — this skill addresses static figure rendering, not interactive analysis

## Evidence

- [intro] Plotting functionality is implemented in Python using spectrum_utils, Matplotlib, and Seaborn.: [evidence span withheld — anchor status: not_found]
