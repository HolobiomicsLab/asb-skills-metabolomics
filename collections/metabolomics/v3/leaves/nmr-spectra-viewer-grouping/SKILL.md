---
name: nmr-spectra-viewer-grouping
description: Use when you have a set of 1D 1H or 13C NMR spectra plus an experimental-design
  table recording factor levels per spectrum, and you need spectra rendered overlaid
  or stacked, colored by factor level.
when_to_use_negative:
- Input is already a binned bucket table destined for statistical analysis rather
  than raw spectra needing visual exploration by factor level
- The task is automated batch-mode spectra processing (baseline correction, ppm calibration,
  alignment, binning) rather than interactive visualization of spectra with factor
  levels
- Spectra lack any associated experimental-design metadata, so no factor-level grouping
  can be applied
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3826
- http://edamontology.org/topic_0092
tools:
- name: NMRProcFlow
  role: Hosts the 'NMR spectra viewer' that lets users visually explore spectra overlaid
    or stacked, zoom on intensity scale, and group spectra by color based on their
    factor level
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Client-server viewer that reads binary spectral data directly and generates
    PNG images (via Gnuplot) of the spectra
- name: R
  role: Language used to load binary 1D spectral intensity data, join spectra IDs
    to factor levels, and render the factor-colored overlay
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:6f8fd6bb989c0faccc01a0ee6280858e7d3466acf75e725624b13086da5d16a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRviewer
  - R
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3826
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nmr-spectra-viewer-grouping

## Summary

Reproduces the NMRProcFlow 'NMR spectra viewer' behavior of grouping 1D NMR spectra by coloring them according to their experimental-factor levels, so that biologists and spectroscopists can visually connect spectral data to the experimental design before downstream statistical analysis.

## When to use

Use when you have a set of 1D 1H or 13C NMR spectra plus an experimental-design table recording factor levels per spectrum, and you need spectra rendered overlaid or stacked, colored by factor level.

## When NOT to use

- Input is already a binned bucket table destined for statistical analysis rather than raw spectra needing visual exploration by factor level
- The task is automated batch-mode spectra processing (baseline correction, ppm calibration, alignment, binning) rather than interactive visualization of spectra with factor levels
- Spectra lack any associated experimental-design metadata, so no factor-level grouping can be applied

## Inputs

- 1D NMR spectral intensity data (1H or 13C) in binary format, as read by NMRviewer
- Experimental-design table mapping each spectrum identifier to its experimental-factor level

## Outputs

- PNG figure of spectra overlaid or stacked, colored by experimental-factor level
- CSV join table mapping spectrum IDs to experimental-factor levels

## How to apply

Load the 1D spectral intensity data into R from binary format, since NMRviewer reads binary data directly and text-to-binary conversion is time-consuming and minimizes data transfers. Read the experimental-design table that records the factor level for each spectrum in the set, then perform a machine-readable join on spectrum identifiers so every spectrum is tagged with its experimental-factor level. Render the spectra overlaid or stacked with an adjustable intensity scale, coloring each spectrum by its factor level; NMRviewer generates PNG images (via Gnuplot) directly from the binary data, and the viewer is the central tool of NMRProcFlow. Export the figure as PNG and save the spectra-ID-to-factor-level join table as CSV for traceability into statistical analyses.

## Related tools

- **NMRProcFlow** (Hosts the 'NMR spectra viewer' that lets users visually explore spectra overlaid or stacked, zoom on intensity scale, and group spectra by color based on their factor level) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client-server viewer that reads binary spectral data directly and generates PNG images (via Gnuplot) of the spectra)
- **R** (Language used to load binary 1D spectral intensity data, join spectra IDs to factor levels, and render the factor-colored overlay)

## Examples

```
sh ./npflow start  # launch NMRProcFlow; in the 'NMR spectra viewer', overlay/stack spectra, zoom the intensity scale, and group spectra by color based on their factor level
```

## Evaluation signals

- Each spectrum in the output PNG carries the color of the factor level assigned by the spectra-ID-to-factor-level join table
- Both the PNG figure and the join-table CSV exist and are non-empty
- Spectra can be toggled between overlaid and stacked rendering with an adjustable intensity scale, as in the NMRProcFlow viewer
- The join table covers every spectrum in the input set with a non-missing factor level after the machine-readable join on spectrum identifiers

## Limitations

- NMRProcFlow's command-line-free GUI exists because R's command-line interface is a genuine barrier for non-expert users; scripted reproduction requires R competence
- Binary spectral data format is required because text-to-binary conversion is time-consuming and data transfers are minimized in NMRviewer
- Spectra alignment, one of the upstream processing steps, requires user expertise and remains an unsolved challenge, which can complicate visual comparison across factor levels

## Evidence

- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
- [abstract] Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels"
- [methods] NMRviewer is based on a client–server architecture: "NMRviewer is based on a client–server architecture"
- [intro] Although an open source software such as R (R Development Core Team 2005), can be leveraged to develop a comprehensive 1D NMR spectra processing: "Although an open source software such as R (R Development Core Team 2005), can be leveraged to develop a comprehensive 1D NMR spectra processing"
