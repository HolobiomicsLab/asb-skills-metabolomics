---
name: spectral-data-frame-io-in-r
description: Use when spectral m/z/intensity data must be loaded into R as plain data
  frames (no S3/S4 masking) to run subMALDI pre-processing such as baselineCorr, or
  to export corrected spectra as two-column text files.
when_to_use_negative:
- Data already lives inside S3/S4 objects from other R MS packages (e.g. MALDIquant,
  MSnbase) and you want to keep working in those class systems rather than plain data
  frames
- The task is peak detection or normalization only, with no need to read/write spectra
  as two-column data frames
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: subMALDI
  role: R package providing baseline correction (monotone_min, linear, loess) and
    data-frame-native spectral handling
  repo: github.com/wesleyburr/subMaldi
- name: R
  role: statistical computing environment in which spectra are read as plain data
    frames and subMALDI is installed from source
- name: MassSpecWavelet
  role: BioConductor dependency that must be installed via BiocManager before the
    subMALDI source install
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:26b268a0e8a6c446fcf773053b815b8f45c233c618ef53835265808b766af515
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spectral-data-frame-io-in-r

## Summary

Reads and writes mass-spectrometry spectra in R as plain two-column data-frame objects (m/z and intensity), so pre-processing like baseline correction via subMALDI leaves data unmasked in S3/S4 classes and interoperable with other R packages.

## When to use

Use when spectral m/z/intensity data must be loaded into R as plain data frames (no S3/S4 masking) to run subMALDI pre-processing such as baselineCorr, or to export corrected spectra as two-column text files.

## When NOT to use

- Data already lives inside S3/S4 objects from other R MS packages (e.g. MALDIquant, MSnbase) and you want to keep working in those class systems rather than plain data frames
- The task is peak detection or normalization only, with no need to read/write spectra as two-column data frames

## Inputs

- bsline/smoothed example spectra files bundled with the subMALDI package (two columns: m/z, intensity)
- raw spectral replicate files readable as two-column data frames (m/z in one column, intensity in another)

## Outputs

- R data-frame objects with m/z and intensity columns, unmasked and accessible after each processing step
- individual text files per corrected spectrum with m/z and corrected intensity columns, as depicted in the vignette figures

## How to apply

Install subMALDI from source: first install BiocManager and the BioConductor dependency MassSpecWavelet, then the CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer and signal, then install the downloaded release tarball (e.g. subMALDI-1.0-2.tar.gz) with type = "source" and dependencies = TRUE. Read the bundled bsline/smoothed example spectra (or your own raw spectra) into R as plain data-frame objects with m/z in one column and intensity in another. Keep the data unmasked — subMALDI is designed so objects stay accessible and compatible with other R packages at every step. For baseline correction, apply the three baselineCorr() methods (monotone_min, linear with n=7, and loess) to the smoothed spectra and subtract the estimated baseline from the raw intensity trace. Write each corrected spectrum to an individual text file with m/z and corrected-intensity columns, matching the layout shown in the vignette figures.

## Related tools

- **subMALDI** (R package providing baseline correction (monotone_min, linear, loess) and data-frame-native spectral handling) — github.com/wesleyburr/subMaldi
- **R** (statistical computing environment in which spectra are read as plain data frames and subMALDI is installed from source)
- **MassSpecWavelet** (BioConductor dependency that must be installed via BiocManager before the subMALDI source install)

## Examples

```
install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE)
```

## Evaluation signals

- Each baseline-corrected spectrum's baseline runs along the x-axis, reproducing the vignette figures
- Raw spectrum's irregularities are removed while peak intensities are retained
- Each exported text file contains exactly two columns (m/z, corrected intensity) matching the vignette layout
- Data remains a plain data frame (no S3/S4 masking) accessible after each processing step
- Source install succeeds only after BiocManager::install("MassSpecWavelet") and the CRAN dependency set are in place

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency
- No DOI or data availability statement exists for the datasets used in the workflow
- The BioConductor install sequence doesn't always seem to work; installing the source tarball with dependencies = TRUE is the workaround
- The tarball filename must be updated to match the version downloaded from the releases folder (e.g. subMALDI-1.0-2.tar.gz)

## Evidence

- [abstract] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects"
- [intro] As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test.: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] install.packages(c("tidyr", "ggplot2", "ggpmisc", "reshape2", "RColorBrewer", "signal")): "install.packages(c("tidyr", "ggplot2", "ggpmisc", "reshape2", "RColorBrewer", "signal"))"
- [readme] install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE): [evidence span withheld — anchor status: external_doc]
