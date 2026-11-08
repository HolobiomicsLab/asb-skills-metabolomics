---
name: mz-intensity-dataframe-manipulation
description: Use when handling irregularly-spaced MALDI-MS or ESI-MS replicate spectra
  as plain R data frames — m/z in column 1, intensities in adjacent columns — for
  blank subtraction, replicate averaging, or pre-processing without S3/S4 wrappers.
when_to_use_negative:
- Input spectra are not on a common m/z grid and have not yet been aligned to the
  array of all theoretically possible m/z values at the specified resolution — align
  first, then subtract
- The user requires S3/S4 object-oriented MS pipelines (e.g. MALDIquant or MSnbase
  class objects); subMALDI's data-frame design is incompatible with such object masks
  by intent
- Blank replicates are unavailable — subSpectra with Blank_Var = 'Average' requires
  an averaged matrix blank column to subtract from sample intensities
tools:
- name: subMALDI
  role: R package providing avgSpectra (mean method) and subSpectra (Blank_Var = 'Average')
    for blank subtraction on m/z-by-intensity data frames
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the data frames are loaded, inspected, and
    validated directly between subMALDI calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:dba4bf94be1fa9ace35f185db3f9606d98c5efd1438efccba24bd489725290cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mz-intensity-dataframe-manipulation

## Summary

Manipulates irregularly-spaced MALDI-MS/ESI-MS spectra as plain R data frames (m/z in the first column, each replicate's intensities in adjacent columns) using subMALDI's avgSpectra, subSpectra, and related functions, avoiding S3/S4 object masking so intensities stay inspectable after every processing step.

## When to use

Use when handling irregularly-spaced MALDI-MS or ESI-MS replicate spectra as plain R data frames — m/z in column 1, intensities in adjacent columns — for blank subtraction, replicate averaging, or pre-processing without S3/S4 wrappers.

## When NOT to use

- Input spectra are not on a common m/z grid and have not yet been aligned to the array of all theoretically possible m/z values at the specified resolution — align first, then subtract
- The user requires S3/S4 object-oriented MS pipelines (e.g. MALDIquant or MSnbase class objects); subMALDI's data-frame design is incompatible with such object masks by intent
- Blank replicates are unavailable — subSpectra with Blank_Var = 'Average' requires an averaged matrix blank column to subtract from sample intensities

## Inputs

- R data frame of aligned spectral replicates: all m/z values in the first column, one intensity column per replicate (e.g. Blank1, Blank2, Before1, Before2), on a common m/z grid at a specified resolution
- Named designation of which columns are matrix blank replicates and which are sample replicates

## Outputs

- Averaged blank intensity column produced by avgSpectra(method = 'mean')
- Blank-subtracted intensity columns (e.g. 'Sub_Before1', 'Sub_Before2') added to the same data frame via subSpectra(Blank_Var = 'Average')
- Final data frame with m/z retained in the first column and blank-subtracted sample intensities in adjacent columns

## How to apply

Load the spectra (e.g. matrix blank replicates 'Blank1'/'Blank2' and sample replicates 'Before1'/'Before2') as data frames with m/z in the first column and intensities in adjacent columns, as produced by subMALDI's alignment step. Because subMALDI deliberately avoids S3/S4 masking, you can inspect, subset, and validate the intensity columns directly with base R at every step. To blank-subtract, first call avgSpectra with method = 'mean' to collapse the blank replicate columns into a single averaged blank column, then call subSpectra with Blank_Var = 'Average' on each sample column, which writes new blank-subtracted columns (e.g. 'Sub_Before1', 'Sub_Before2') into the same data frame. Confirm the output frame retains m/z in the first column with each sample's subtracted intensities in adjacent columns, matching the vignette output before saving.

## Related tools

- **subMALDI** (R package providing avgSpectra (mean method) and subSpectra (Blank_Var = 'Average') for blank subtraction on m/z-by-intensity data frames) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the data frames are loaded, inspected, and validated directly between subMALDI calls)

## Examples

```
avg <- avgSpectra(df, method = "mean"); sub <- subSpectra(avg, Blank_Var = "Average")
```

## Evaluation signals

- Output data frame retains m/z values in the first column, with each sample's blank-subtracted intensities in adjacent columns (schema invariant)
- Averaged blank column equals the column-wise mean of Blank1 and Blank2 intensities (method = 'mean')
- Each 'Sub_*' column equals the original sample column minus the averaged blank column, element-wise on the shared m/z grid
- Subtracted columns reproduce the blank-subtracted intensities shown in the subMALDI vignette output for Before1/Before2
- m/z values remain unchanged by subtraction — only intensity columns are modified

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency, so reproduction may require pinning from the GitHub releases folder (e.g. subMALDI-1.0-2.tar.gz)
- No DOI or data availability statement is given for the workflow datasets, so vignette inputs must be obtained from the package/vignette itself
- Installing from source requires the BioConductor dependency MassSpecWavelet plus CRAN packages (tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal), and the BioConductor install sequence does not always work as documented

## Evidence

- [abstract] The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns: "The resulting data frame contains all m/z data
in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [abstract] permits organization, pre-processing, and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects"
- [discussion] These replicates can then be normalized, averaged, subtracted, and plotted using subMALDI's suite of functions: "These replicates can then be normalized, averaged, subtracted, and plotted using `subMALDI`'s suite of functions"
- [readme] subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test.: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] begin by downloading the source copy of the package from the releases folder: "begin by downloading the source copy of the package from the [releases folder](https://github.com/wesleyburr/subMaldi/releases)"
