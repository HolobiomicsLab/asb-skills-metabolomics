---
name: spectra-metadata-factor-level-join
description: Use when 1D NMR spectral intensity data must be annotated with experimental-design
  metadata — i.e. spectrum IDs need to be tagged with factor levels so spectra can
  be grouped and colored by condition in an overlay or stacked viewer figure.
when_to_use_negative:
- Input is already a processed bucket/feature table (post baseline correction, alignment
  and binning) rather than per-spectrum 1D intensity traces to be visualized
- No experimental-design metadata (factor levels per spectrum) exists, so there is
  nothing to join on and no color grouping to reproduce
- The goal is the spectra processing itself (ppm-scale calibration, baseline correction,
  realignment, binning) rather than visualization and design annotation
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: NMRProcFlow
  role: 'Reference open source GUI whose ''NMR spectra viewer'' central tool defines
    the behavior being reproduced: spectra overlaid or stacked, zoom on intensity
    scale, and grouping sets of spectra by color based on their factor level'
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: 'Client-server spectra viewer whose implementation pattern is followed: reading
    1D intensity data in binary format and generating PNG images directly from the
    binary data'
- name: R
  role: Implementation language for loading the binary spectral data, performing the
    spectrum-ID-to-factor-level join, and rendering/exporting the colored figure and
    CSV join table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:b5b7c13f3010fe1662b0bb15fedc9121f35a36f090567b1175bdf51cd68b2add
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRviewer
  - R
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spectra-metadata-factor-level-join

## Summary

Reproduces NMRProcFlow's 'NMR spectra viewer' behavior of grouping 1D NMR spectra by coloring them according to their experimental factor levels, by machine-readable joining spectrum identifiers to the experimental-design table and rendering colored overlay/stacked spectral figures.

## When to use

Use when 1D NMR spectral intensity data must be annotated with experimental-design metadata — i.e. spectrum IDs need to be tagged with factor levels so spectra can be grouped and colored by condition in an overlay or stacked viewer figure.

## When NOT to use

- Input is already a processed bucket/feature table (post baseline correction, alignment and binning) rather than per-spectrum 1D intensity traces to be visualized
- No experimental-design metadata (factor levels per spectrum) exists, so there is nothing to join on and no color grouping to reproduce
- The goal is the spectra processing itself (ppm-scale calibration, baseline correction, realignment, binning) rather than visualization and design annotation

## Inputs

- 1D NMR spectral intensity data (e.g. 1H or 13C), preferably in binary format as read by NMRviewer
- Experimental-design table recording the factor level for each spectrum in the set
- Spectrum identifiers shared between the intensity data and the design table

## Outputs

- PNG figure of spectra overlaid or stacked, each colored by its experimental factor level (with adjustable intensity scale)
- CSV join table mapping each spectrum ID to its experimental factor level

## How to apply

Load the 1D spectral intensity data into R, reading it in binary format as NMRviewer does, since text-to-binary conversion is time consuming and data transfers are minimized. Read the experimental-design table recording the factor level for each spectrum, then perform a machine-readable join on spectrum identifiers so every spectrum is tagged with its experimental-factor level — this reproduces the spectra-viewer's factor-grouping mechanism rather than ad hoc visual labeling. Render the spectra overlaid or stacked with an adjustable intensity scale, coloring each spectrum by its factor level, mirroring NMRProcFlow's viewer in which images are generated in PNG format directly from the binary data. Export the rendered figure as PNG and write the spectrum-ID-to-factor-level join table as CSV; decision point: choose overlaid vs. stacked rendering and the intensity-scale zoom before export based on how clearly the factor-level color grouping must be discernible.

## Related tools

- **NMRProcFlow** (Reference open source GUI whose 'NMR spectra viewer' central tool defines the behavior being reproduced: spectra overlaid or stacked, zoom on intensity scale, and grouping sets of spectra by color based on their factor level) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client-server spectra viewer whose implementation pattern is followed: reading 1D intensity data in binary format and generating PNG images directly from the binary data)
- **R** (Implementation language for loading the binary spectral data, performing the spectrum-ID-to-factor-level join, and rendering/exporting the colored figure and CSV join table)

## Examples

```
Rscript -e 'j <- merge(read.csv("spectra_ids.csv"), read.csv("design.csv"), by = "spectrum_id"); write.csv(j, "spectra_factor_join.csv", row.names = FALSE)'
```

## Evaluation signals

- Every spectrum in the output PNG carries the color of the factor level assigned by the join table
- Both the PNG figure and the join-table CSV exist and are non-empty
- Each spectrum identifier in the intensity data appears exactly once in the spectrum-ID-to-factor-level join table (complete, unambiguous join)
- The rendered figure supports overlaid/stacked display with an adjustable (zoomable) intensity scale, mirroring the NMR spectra viewer
- The figure is generated directly from the binary intensity data rather than re-encoded text, matching the NMRviewer implementation rationale

## Limitations

- Text-to-binary conversion of spectral data is time consuming, so the workflow expects binary-format intensity data; text inputs add conversion overhead
- The viewer behavior applies to 1D NMR spectra (1H and 13C); 2D or non-NMR data are out of scope
- Deploying NMRProcFlow itself requires a recent Linux OS supporting Docker, with the 'npflow' image built or pulled from docker hub
- Fully automated batch-mode spectra processing workflows usable by non-experts have not reached full maturity; the interactive viewer remains the reference behavior being emulated

## Evidence

- [readme] readme: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
- [abstract] abstract: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels"
- [methods] methods: "NMRviewer is based on a client–server architecture"
