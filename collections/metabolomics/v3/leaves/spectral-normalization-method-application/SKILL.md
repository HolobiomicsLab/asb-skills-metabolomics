---
name: spectral-normalization-method-application
description: Use when replicate MALDI-MS/ESI-MS spectra stored as irregularly-spaced
  m/z–intensity data frames must be placed on a comparable intensity scale before
  averaging, subtraction, or plotting.
when_to_use_negative:
- The input is already normalized intensity data or a downstream feature/peak table
  rather than raw replicate spectra needing intensity rescaling
- The task is another pre-processing stage (smoothing, baseline correction, peak detection)
  rather than normalization — use the corresponding subMALDI step instead
- The spectra are held in S3/S4 class objects from other packages (e.g. MALDIquant
  or MSnbase) and the workflow requires that object model rather than subMALDI's accessible
  data-frame format
edam_operation: http://edamontology.org/operation_3435
tools:
- name: subMALDI
  role: Provides the normSpectra function implementing the nine normalization methods,
    including rel_TIC, on irregularly-spaced spectral data frames
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which subMALDI's normSpectra is installed (from
    source, with MassSpecWavelet via BiocManager and CRAN packages tidyr, ggplot2,
    ggpmisc, reshape2, RColorBrewer, signal) and invoked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:03f26341f9dff0b1be300a698a515d7a4710ebdfbec0ba286c2e9df7e0724be3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# spectral-normalization-method-application

## Summary

Applies the subMALDI R package's normSpectra function to normalize mass spectral replicate intensities using one of nine methods (custom, max, max_set, RMS, TIC, median, stdev, quantile, rel_TIC), operating on irregularly-spaced spectra held in plain data frames. Relative-TIC normalization rescales each spectrum's total ion current to 1 so a set of n spectra sums to n.

## When to use

Use when replicate MALDI-MS/ESI-MS spectra stored as irregularly-spaced m/z–intensity data frames must be placed on a comparable intensity scale before averaging, subtraction, or plotting.

## When NOT to use

- The input is already normalized intensity data or a downstream feature/peak table rather than raw replicate spectra needing intensity rescaling
- The task is another pre-processing stage (smoothing, baseline correction, peak detection) rather than normalization — use the corresponding subMALDI step instead
- The spectra are held in S3/S4 class objects from other packages (e.g. MALDIquant or MSnbase) and the workflow requires that object model rather than subMALDI's accessible data-frame format

## Inputs

- subMALDI spectral data frame: m/z values in the first column, one intensity column per spectral replicate (e.g. Before1, Before2 sample spectra)
- Normalization method name (one of custom, max, max_set, RMS, TIC, median, stdev, quantile, rel_TIC)
- For rel_TIC: the target total, set to the number of spectra in the set

## Outputs

- One normalized spectral data frame per normalization method, with the same m/z column and rescaled intensity columns
- For rel_TIC: verification that each spectrum's total ion current (column sum) equals exactly 1 and the set's total equals the number of spectra

## How to apply

Load the spectra (e.g. the Before1/Before2 sample data shipped with subMALDI) into a single spectral data frame with m/z in the first column and each replicate's intensities in adjacent columns, as subMALDI's alignment step produces. Call normSpectra specifying the method: 'rel_TIC' with the total set equal to the number of spectra in the set rescales each spectrum's total ion current to exactly 1; 'custom' accepts a user-defined scaling, and 'max', 'max_set', 'RMS', 'TIC', 'median', 'stdev', and 'quantile' each produce their own normalized data frame. Run normSpectra once per method to generate one normalized data frame per approach, since subMALDI deliberately avoids S3/S4 masking so the m/z and intensity values stay accessible and directly comparable between methods. Verify the rel_TIC result by computing each normalized spectrum's column sum and confirming every TIC equals 1 and the summed intensities equal the number of spectra in the set.

## Related tools

- **subMALDI** (Provides the normSpectra function implementing the nine normalization methods, including rel_TIC, on irregularly-spaced spectral data frames) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which subMALDI's normSpectra is installed (from source, with MassSpecWavelet via BiocManager and CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal) and invoked)

## Examples

```
normSpectra(spectra, method = "rel_TIC", total = ncol(spectra) - 1); colSums(normalized[, -1])
```

## Evaluation signals

- After rel_TIC normalization, the column sum (TIC) of every normalized spectrum equals exactly 1
- The summed intensities across all rel_TIC-normalized spectra equal the number of spectra in the set (e.g. Before1 + Before2 sum to 2)
- Running normSpectra with each of the nine methods yields a distinct normalized data frame per method, all sharing the identical m/z column
- The normalized data frame retains the subMALDI layout: all m/z values in the first column with each replicate's intensities in adjacent columns, with no S3/S4 object masking
- Because data objects stay unmasked, intermediate values can be inspected with ordinary R functions after each normalization step

## Limitations

- The normalization operates on subMALDI's unmasked data-frame layout; spectra in S3/S4 objects from other pipelines must first be converted
- No specific parameter values, version numbers, or per-method thresholds for normSpectra are stated in the article or README; method-specific options must be checked in the package vignettes
- subMALDI is designed for irregularly-spaced spectral data aligned to a theoretical m/z grid at a specified resolution; spectra already on a regular grid from other tools may need re-alignment first

## Evidence

- [abstract] subMALDI permits direct comparison of irregularly spaced spectral replicates without S3/S4 masking: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects"
- [abstract] Normalized output keeps m/z in the first column with replicate intensities adjacent: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [readme] README confirms subMALDI provides normalization of spectral data sets in R: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [intro] subMALDI data objects stay compatible with other R packages and user-defined functions: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
