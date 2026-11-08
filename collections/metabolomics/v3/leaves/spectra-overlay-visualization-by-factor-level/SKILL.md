---
name: spectra-overlay-visualization-by-factor-level
description: Use when 1D NMR spectral intensity data (e.g. Bruker 1r-derived spectra)
  must be visualized grouped or colored by experimental-factor levels, bridging experimental
  design and downstream statistical analyses.
when_to_use_negative:
- Input is already a processed feature/bucket table rather than 1D spectral intensities
  — this skill renders spectra, not bucketed data.
- The goal is spectra processing (ppm-scale calibration, baseline correction, realignment,
  binning) rather than factor-level visualization — use NMRProcFlow's processing workflow
  instead.
- An interactive GUI session is needed for expert inspection of intertwined peaks;
  NMRProcFlow's command-line-free GUI is designed for that, whereas this skill produces
  static PNG output.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0523
- http://edamontology.org/topic_3318
tools:
- name: NMRProcFlow
  role: Reference implementation whose 'NMR spectra viewer' the factor-level coloring
    and overlaid/stacked rendering reproduce
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Precedent for reading 1D spectral intensities in binary format to minimize
    data transfer and avoid slow text-to-binary conversion
- name: R
  role: Language used to load the binary intensity data, join spectra IDs to factor
    levels, and render/export the colored figure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:1fba99ba8ccef2cc22242978328931ba083b7609f3ed16352563b1a6b40e12cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRviewer
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0523
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectra-overlay-visualization-by-factor-level

## Summary

Reproduces NMRProcFlow's 'NMR spectra viewer' behavior outside the GUI: load 1D NMR spectral intensities (in the binary format NMRviewer reads, since text-to-binary conversion is slow), join each spectrum to its experimental-factor level from the design table, and render overlaid/stacked spectra with per-factor-level colors as a PNG figure plus a machine-readable join table.

## When to use

Use when 1D NMR spectral intensity data (e.g. Bruker 1r-derived spectra) must be visualized grouped or colored by experimental-factor levels, bridging experimental design and downstream statistical analyses.

## When NOT to use

- Input is already a processed feature/bucket table rather than 1D spectral intensities — this skill renders spectra, not bucketed data.
- The goal is spectra processing (ppm-scale calibration, baseline correction, realignment, binning) rather than factor-level visualization — use NMRProcFlow's processing workflow instead.
- An interactive GUI session is needed for expert inspection of intertwined peaks; NMRProcFlow's command-line-free GUI is designed for that, whereas this skill produces static PNG output.

## Inputs

- 1D NMR spectral intensity data in binary format (e.g. as read by NMRviewer)
- Experimental-design table mapping each spectrum identifier to its experimental-factor level

## Outputs

- PNG figure of overlaid/stacked spectra colored by experimental-factor level
- CSV join table of spectrum ID to experimental-factor level

## How to apply

Load the 1D spectral intensity data into R, reading the spectral data in binary format the way NMRviewer does, since text-to-binary conversion is time consuming and data transfers are minimized. Read the experimental-design table recording the factor level of each spectrum, then perform a machine-readable join on spectrum identifiers so every spectrum is tagged with its experimental-factor level. Render the spectra overlaid or stacked with an adjustable intensity scale, coloring each spectrum by its factor level, mirroring NMRProcFlow's viewer in which PNG images are generated directly from the binary data. Export the rendered figure as PNG and save the spectra-ID-to-factor-level join table as CSV so the coloring can be verified against the design metadata.

## Related tools

- **NMRProcFlow** (Reference implementation whose 'NMR spectra viewer' the factor-level coloring and overlaid/stacked rendering reproduce) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Precedent for reading 1D spectral intensities in binary format to minimize data transfer and avoid slow text-to-binary conversion)
- **R** (Language used to load the binary intensity data, join spectra IDs to factor levels, and render/export the colored figure)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- Every spectrum in the output PNG carries the color of the factor level assigned by the join table.
- Both the PNG figure and the spectra-ID-to-factor-level join-table CSV exist and are non-empty.
- The join of spectrum identifiers against the design table is complete: no spectrum in the intensity matrix is left untagged with a factor level.
- The rendered plot supports both overlaid and stacked layouts with an adjustable intensity scale, matching the viewer's described behavior.

## Limitations

- NMRviewer/NMRProcFlow use a client-server architecture (Docker images communicating via AJAX web services), so fully standalone reproduction must reimplement the binary-format reading of intensities.
- The output is a static PNG (generated via Gnuplot in the original), losing the interactive zoom and group-by-color reassignment of the GUI viewer.
- The workflow depends on the experimental-design table correctly recording the factor level for each spectrum; mislabeled design metadata propagates directly to the coloring.

## Evidence

- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
- [methods] NMRviewer is based on a client–server architecture: "NMRviewer is based on a client–server architecture"
- [methods] NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R: "NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R"
- [abstract] Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels"
- [full_text] as NMRviewer does, since text-to-binary conversion is time consuming and data transfers are minimized: [evidence span withheld — anchor status: not_found]
