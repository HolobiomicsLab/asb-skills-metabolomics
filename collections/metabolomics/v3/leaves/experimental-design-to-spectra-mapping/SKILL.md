---
name: experimental-design-to-spectra-mapping
description: Use when you have 1D NMR spectral intensity data plus an experimental-design
  table recording factor levels per spectrum, and need to visually map design metadata
  onto spectra (e.g. colored overlays) or export the spectrum-to-factor join for statistics.
when_to_use_negative:
- Input is already a processed feature/bucket table keyed by samples rather than raw
  1D spectra needing design overlay — the factor grouping here operates on spectra
  in the viewer.
- The task is spectra processing itself (ppm-scale calibration, baseline correction,
  realignment of ppm areas, binning) rather than design-to-spectra annotation.
- Only 2D NMR or non-vendor-FID formats not accepted by NMRProcFlow (e.g. no Bruker/Agilent
  FID or nmrML) are available without conversion.
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3170
tools:
- name: NMRProcFlow
  role: Reference implementation whose 'NMR spectra viewer' groups sets of spectra
    by color based on their factor level; the skill reproduces that behavior programmatically.
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Client-server spectra viewer whose approach of reading spectral data in binary
    format (instead of text) the R join/rendering steps follow.
- name: R
  role: Language used to load the binary 1D spectral intensities, join spectrum IDs
    to factor levels, and render the colored overlay.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:4e5da538a12f55750504515f1e49c93bff45a708e6a907174897d7b8d6ff73d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRviewer
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# experimental-design-to-spectra-mapping

## Summary

Reproduces NMRProcFlow's 'NMR spectra viewer' factor-grouping behavior by joining each 1D NMR spectrum identifier to its experimental-factor level from the design table, then rendering the spectra overlaid or stacked with factor-level colors. This bridges experimental design metadata and downstream statistical analyses.

## When to use

Use when you have 1D NMR spectral intensity data plus an experimental-design table recording factor levels per spectrum, and need to visually map design metadata onto spectra (e.g. colored overlays) or export the spectrum-to-factor join for statistics.

## When NOT to use

- Input is already a processed feature/bucket table keyed by samples rather than raw 1D spectra needing design overlay — the factor grouping here operates on spectra in the viewer.
- The task is spectra processing itself (ppm-scale calibration, baseline correction, realignment of ppm areas, binning) rather than design-to-spectra annotation.
- Only 2D NMR or non-vendor-FID formats not accepted by NMRProcFlow (e.g. no Bruker/Agilent FID or nmrML) are available without conversion.

## Inputs

- 1D NMR spectral intensity data in binary format (e.g. Bruker 1r-derived binary matrices as read by NMRviewer)
- Experimental-design table recording the experimental factor level for each spectrum identifier (CSV/TSV)
- Mapping of spectrum identifiers used consistently in both the spectral data and the design table

## Outputs

- PNG figure of spectra overlaid or stacked, colored by experimental factor level (Gnuplot-generated)
- CSV join table mapping each spectrum identifier to its experimental factor level

## How to apply

Load the 1D spectral intensity data into R, reading spectra in binary format as NMRviewer does, since text-to-binary conversion is time consuming and this minimizes data transfers. Read the experimental-design table that records the factor level for each spectrum, then perform a machine-readable join on spectrum identifiers so every spectrum is tagged with its factor level. Render the spectra overlaid or stacked with an adjustable intensity scale, coloring each spectrum by its factor level, mirroring NMRProcFlow's viewer in which PNG images are generated via Gnuplot directly from the binary data. Export the rendered figure as PNG and save the spectra-ID-to-factor-level join table as CSV for downstream statistical analyses.

## Related tools

- **NMRProcFlow** (Reference implementation whose 'NMR spectra viewer' groups sets of spectra by color based on their factor level; the skill reproduces that behavior programmatically.) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client-server spectra viewer whose approach of reading spectral data in binary format (instead of text) the R join/rendering steps follow.)
- **R** (Language used to load the binary 1D spectral intensities, join spectrum IDs to factor levels, and render the colored overlay.)

## Examples

```
Rscript overlay_factors.R --spectra spectra.bin --design design.csv --join join.csv --out spectra_by_factor.png
```

## Evaluation signals

- Every spectrum in the output PNG carries the color of the factor level assigned by the join table.
- Both the PNG figure and the spectra-ID-to-factor-level join-table CSV exist and are non-empty.
- Row count of the join-table CSV equals the number of spectra in the loaded binary data set (one tagged factor level per spectrum).
- Rendering supports overlaid or stacked display with adjustable intensity scale, matching the viewer's stated capabilities.
- Spectrum identifiers in the CSV match identifiers present in the experimental-design table (join produced no orphan spectra).

## Limitations

- Reading spectra as text instead of binary is time consuming and increases data transfers; the binary approach (as NMRviewer does) is preferred.
- Images in the viewer are generated in PNG format via Gnuplot directly from binary data, so the visualization fidelity depends on that rendering path.
- Interactive processing with a spectra viewer is still considered the best approach; automated batch-mode 1D NMR processing workflows have not reached full maturity.

## Evidence

- [abstract] visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between"
- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application. It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to"
- [methods] NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R: "NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R"
