---
name: maldi-intensity-data-frame-organization
description: Use when raw MALDI-MS or ESI-MS spectra with mismatched, irregularly-spaced
  m/z grids must be organized into one m/z-by-replicate data frame before smoothing,
  baseline correction, peak detection, or normalization (e.g., rel_TIC).
when_to_use_negative:
- Data are already aligned on a common m/z grid and stored in a tidy table — re-alignment
  is unnecessary
- The workflow requires S3/S4 class-based pipelines (e.g., MSnbase or MALDIquant object
  models) rather than accessible data frames
- Data are not mass spectrometry spectra with m/z and intensity columns
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: subMALDI
  role: R package whose organization/alignment functions build the m/z-by-replicate
    data frame at a specified resolution
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which subMALDI runs and the data frame is manipulated
- name: MassSpecWavelet
  role: BioConductor dependency that must be installed via BiocManager before subMALDI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:9aa4967eb0129f24d23d06664635dc9de32ed1abb1788ab711ba3d69042e17da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# maldi-intensity-data-frame-organization

## Summary

Organizes irregularly-spaced MALDI-MS/ESI-MS spectra into a single rectangular data frame using the subMALDI R package, with all m/z values in the first column and each spectral replicate's intensities in adjacent columns, without hiding data in S3/S4 objects.

## When to use

Use when raw MALDI-MS or ESI-MS spectra with mismatched, irregularly-spaced m/z grids must be organized into one m/z-by-replicate data frame before smoothing, baseline correction, peak detection, or normalization (e.g., rel_TIC).

## When NOT to use

- Data are already aligned on a common m/z grid and stored in a tidy table — re-alignment is unnecessary
- The workflow requires S3/S4 class-based pipelines (e.g., MSnbase or MALDIquant object models) rather than accessible data frames
- Data are not mass spectrometry spectra with m/z and intensity columns

## Inputs

- Raw or processed MALDI-MS / ESI-MS spectra with irregularly-spaced m/z values (e.g., subMALDI's Before1 and Before2 sample spectra)
- A target mass range and m/z resolution for grid alignment

## Outputs

- A single R data frame with all m/z values in the first column and each spectral replicate's intensity values in adjacent columns
- An intermediate organization step feeding subMALDI pre-processing (smoothing, baseline correction, peak detection) and normalization (e.g., normSpectra with rel_TIC)

## How to apply

Install subMALDI from source after first installing the BioConductor dependency MassSpecWavelet via BiocManager, plus the CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, and signal. Load the raw spectra (e.g., the package's Before1/Before2 sample spectra) and organize them into a single spectral data frame. subMALDI aligns processed spectral intensity values to an array of all theoretically possible m/z values in the observed mass range at a user-specified resolution, so choose the resolution to match the instrument. The resulting data frame keeps all m/z data in the first column with each replicate's intensities in adjacent columns, leaving m/z and intensity values accessible after every processing step for use with other R packages or user-defined functions.

## Related tools

- **subMALDI** (R package whose organization/alignment functions build the m/z-by-replicate data frame at a specified resolution) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which subMALDI runs and the data frame is manipulated)
- **MassSpecWavelet** (BioConductor dependency that must be installed via BiocManager before subMALDI)

## Examples

```
install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE)
```

## Evaluation signals

- The output is a data frame (not an S3/S4 object) whose first column contains all m/z values and whose adjacent columns each hold one spectral replicate's intensities
- All replicate spectra occupy a shared m/z grid of all theoretically possible m/z values in the observed mass range at the specified resolution — no irregularly-spaced residuals remain
- m/z and intensity values remain directly accessible after each processing step (they are not masked by class wrappers)
- Downstream subMALDI steps (e.g., normSpectra with method 'rel_TIC') accept the data frame without further reshaping

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency
- The BioConductor installation sequence 'doesn't always seem to work'; installing the source tarball with dependencies = TRUE is the recommended fallback
- Package must be installed from the releases folder as a source tarball (e.g., subMALDI-1.0-2.tar.gz); filename must match the downloaded version

## Evidence

- [abstract] data frame with all m/z in first column: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [abstract] no S3/S4 masking: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects"
- [abstract] alignment to m/z grid: "aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution"
- [intro] compatibility with other R packages: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] installation dependencies: "install.packages(c("tidyr", "ggplot2", "ggpmisc", "reshape2", "RColorBrewer", "signal"))"
