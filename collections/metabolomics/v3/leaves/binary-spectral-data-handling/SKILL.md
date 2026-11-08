---
name: binary-spectral-data-handling
description: Use when 1D NMR spectral intensities are supplied in binary format (e.g.,
  Bruker 1r files) and spectra must be loaded in R, tagged with experimental-factor
  levels, and visualized as overlaid/stacked colored traces.
when_to_use_negative:
- Input is already a processed feature/bucket table rather than raw 1D spectral intensity
  data
- The task is purely text-based handling where binary conversion cost is irrelevant
  and datasets are small
- A non-expert needs fully automatic batch-mode processing with no interactive spectra
  viewing — the article states such workflows 'have not yet reached full maturity'
edam_operation: http://edamontology.org/operation_3357
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRProcFlow
  role: GUI application whose central 'NMR spectra viewer' displays spectra overlaid
    or stacked with coloring by factor level, connecting experimental design to downstream
    statistical analyses
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Client-server component that reads spectral data in binary format and generates
    PNG images via Gnuplot directly from the binary data
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Implementation language for NMRspec/NMRProcFlow; used to load binary spectral
    data and join spectra to factor levels in a reusable script
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: Deployment container for NMRProcFlow modules (npflow image) that share a data
    volume and communicate via AJAX web services
  repo: https://github.com/INRA/NMRProcFlow
- name: Rcpp
  role: C++ acceleration layer for some processing algorithms on the binary spectral
    data
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:4855db807f90bfd8a8cc781bf77be70fe194daaaf2168cdee1d108a8b52c78b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - NMRviewer
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3357
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# binary-spectral-data-handling

## Summary

Handles 1D NMR spectral intensity data stored in binary format, as NMRviewer does, so that large sets of spectra can be loaded, joined to experimental-design metadata, and rendered (e.g., to PNG via Gnuplot) without costly text-to-binary conversion.

## When to use

Use when 1D NMR spectral intensities are supplied in binary format (e.g., Bruker 1r files) and spectra must be loaded in R, tagged with experimental-factor levels, and visualized as overlaid/stacked colored traces.

## When NOT to use

- Input is already a processed feature/bucket table rather than raw 1D spectral intensity data
- The task is purely text-based handling where binary conversion cost is irrelevant and datasets are small
- A non-expert needs fully automatic batch-mode processing with no interactive spectra viewing — the article states such workflows 'have not yet reached full maturity'

## Inputs

- 1D NMR spectral intensity data in binary format (e.g., Bruker 1r files from Bruker or Agilent/Varian FID-derived data)
- Experimental-design table recording the factor level for each spectrum

## Outputs

- PNG figure of spectra overlaid or stacked, colored by experimental-factor level
- CSV join table mapping spectrum identifiers to experimental-factor levels

## How to apply

Load the 1D spectral intensity data directly in binary format, since text-to-binary conversion is time consuming and binary transfer minimizes data movement — the same approach NMRviewer uses, generating PNG images via Gnuplot directly from the binary data. Read the experimental-design table recording the factor level for each spectrum, then perform a machine-readable join on spectrum identifiers so every spectrum is tagged with its experimental-factor level. Render the spectra overlaid or stacked with an adjustable intensity scale, coloring each spectrum by its factor level to mirror the NMRProcFlow 'NMR spectra viewer'. Export the figure as PNG and save the spectrum-ID-to-factor-level join table as CSV. Where processing is needed (ppm scale calibration, baseline correction locally or fully, realignment of ppm areas, binning), NMRProcFlow's interactive spectra viewer remains the core tool because fully automatic batch processing is still not mature for non-expert users.

## Related tools

- **NMRProcFlow** (GUI application whose central 'NMR spectra viewer' displays spectra overlaid or stacked with coloring by factor level, connecting experimental design to downstream statistical analyses) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client-server component that reads spectral data in binary format and generates PNG images via Gnuplot directly from the binary data) — https://github.com/INRA/NMRProcFlow
- **R** (Implementation language for NMRspec/NMRProcFlow; used to load binary spectral data and join spectra to factor levels in a reusable script) — https://github.com/INRA/NMRProcFlow
- **Docker** (Deployment container for NMRProcFlow modules (npflow image) that share a data volume and communicate via AJAX web services) — https://github.com/INRA/NMRProcFlow
- **Rcpp** (C++ acceleration layer for some processing algorithms on the binary spectral data) — https://github.com/INRA/NMRProcFlow

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow
```

## Evaluation signals

- Every spectrum in the output PNG carries the color of the factor level assigned by the join table
- Both the PNG figure and the spectra-ID-to-factor-level join-table CSV exist and are non-empty
- The binary data are read without a text-to-binary conversion step, matching NMRviewer's design rationale
- Spectra render as overlaid or stacked traces with a user-adjustable intensity scale, as in the NMR spectra viewer
- If processing is applied, ppm-scale calibration, baseline correction, realignment, and binning outputs remain consistent with NMRProcFlow's stated processing steps

## Limitations

- NMRviewer uses a client-server architecture with modules in Docker images communicating via AJAX web services, so a Docker-capable Linux host is required
- Text-to-binary conversion of spectral data is time consuming, so workflows that insist on text interchange will be slow at scale
- Fully automatic batch-mode 1D NMR processing usable by non-experts has not reached full maturity; interactive inspection remains necessary
- Spectra alignment, where resonance chemical shifts vary uncontrolled between spectra, requires user expertise and is not fully solved

## Evidence

- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
- [methods] NMRviewer is based on a client–server architecture: "NMRviewer is based on a client–server architecture"
- [methods] some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011): "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
- [intro] the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode: "the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode"
- [readme] the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
