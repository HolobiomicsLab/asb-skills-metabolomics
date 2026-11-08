---
name: batch-directory-spectrum-import
description: Use when the starting data are many single-spectrum .CSV files (one spectrum
  per file) or raw .ascii instrument output, and each spectrum must be converted into
  a pairwise m/z/intensity data frame — including batch conversion of a whole folder
  into .rda files.
when_to_use_negative:
- Spectra are already available in a structured interchange format (e.g. mzML) that
  other R packages such as MALDIquant or MSnbase can read directly
- Only a single spectrum needs importing and no batch directory conversion is required
  — use the single-spectrum import path instead of the directory batch function
- The spectra are already stored as .rda data frames from a previous import run
edam_operation: http://edamontology.org/operation_3222
tools:
- name: subMALDI
  role: Provides the import functions readcsvSpec() (single .CSV spectrum), read_ascii()
    (.ascii instrument output), and readcsvDir() (batch directory conversion to .rda
    files)
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the import functions are called and the pairwise
    m/z/intensity data frames are produced
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:a2657baa810ccdc18b4b1865f9c961f9fe577ba2d3bb876cdf3d5ccf43883d51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# batch-directory-spectrum-import

## Summary

Imports single-spectrum .CSV files and raw .ascii instrument output into pairwise m/z/intensity data frames, and batch-converts an entire directory of .CSV spectra into .rda files using subMALDI's readcsvSpec, read_ascii, and readcsvDir functions in R.

## When to use

Use when the starting data are many single-spectrum .CSV files (one spectrum per file) or raw .ascii instrument output, and each spectrum must be converted into a pairwise m/z/intensity data frame — including batch conversion of a whole folder into .rda files.

## When NOT to use

- Spectra are already available in a structured interchange format (e.g. mzML) that other R packages such as MALDIquant or MSnbase can read directly
- Only a single spectrum needs importing and no batch directory conversion is required — use the single-spectrum import path instead of the directory batch function
- The spectra are already stored as .rda data frames from a previous import run

## Inputs

- Single-spectrum .CSV files (one spectrum per file) in an input directory
- Raw .ascii instrument output files

## Outputs

- Two-column (m/z, intensity) pairwise data frames, one per imported spectrum
- One .rda file per spectrum written to a designated output folder (from readcsvDir)

## How to apply

In R with the subMALDI package (source-installed from the GitHub releases folder, with the BioConductor dependency MassSpecWavelet and CRAN packages such as tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, and signal installed first), use readcsvSpec() to import one single-spectrum .CSV file at a time, returning a data frame of pairwise m/z and intensity values. Use read_ascii() when the input is raw .ascii instrument output, converting it to a data.frame object. When many spectra must be ingested at once, use readcsvDir(), which applies the CSV import mechanism across all .CSV files in a folder and writes the resulting pairwise data frames out as .rda files into a designated output folder. subMALDI deliberately avoids masking data into S3 or S4 class objects, so the imported m/z and intensity data remain plain, accessible data frames compatible with other R packages and downstream subMALDI pre-processing (smoothing, baseline correction, peak detection, normalization).

## Related tools

- **subMALDI** (Provides the import functions readcsvSpec() (single .CSV spectrum), read_ascii() (.ascii instrument output), and readcsvDir() (batch directory conversion to .rda files)) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the import functions are called and the pairwise m/z/intensity data frames are produced)

## Evaluation signals

- Each imported file yields exactly a two-column data frame: m/z values in the first column and the corresponding intensities in the adjacent column
- readcsvDir() produces exactly one .rda file per spectrum present in the input directory
- Each .rda file loads back (readRDS/load) as a data frame with the same pairwise m/z/intensity structure as the in-memory import
- .ascii instrument output is returned as a data.frame object rather than a raw text or S3/S4 object
- The package installs cleanly from source with dependencies = TRUE, confirming the MassSpecWavelet BioConductor dependency and CRAN packages are present

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency, so the import behavior may vary across releases
- No changelog or version history is provided for the package
- The README notes that the BioConductor install sequence for MassSpecWavelet 'doesn't always seem to work' unless dependencies = TRUE is set during source installation

## Evidence

- [abstract] Data frame column layout: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [intro] Package design keeps data accessible: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] CRAN dependencies for installation: "install.packages(c("tidyr", "ggplot2", "ggpmisc", "reshape2", "RColorBrewer", "signal"))"
- [intro] Open framework scope: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
