---
name: batch-normalization-comparison-across-methods
description: Use when raw or pre-processed MALDI/ESI-MS replicate intensities sit
  in an R data frame (m/z column plus per-replicate intensity columns) and you must
  normalize them and/or compare which normalization scheme to adopt.
when_to_use_negative:
- Input intensities are already normalized (e.g. prior rel_TIC/TIC scaling applied)
  — re-normalizing will distort the relative scale
- Data are held as S3/S4 class objects from other MS packages (e.g. MALDIquant, MSnbase)
  rather than plain data frames — extract m/z/intensity columns into a data frame
  first, since subMALDI operates on unmasked data objects
- The task is peak detection or baseline correction rather than intensity scaling
  — use subMALDI's pre-processing functions instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: subMALDI
  role: Provides normSpectra and the nine normalization methods; the package's data-frame
    design keeps m/z and intensity data accessible after each normalization step
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Runtime environment in which subMALDI normalization is executed; requires
    MassSpecWavelet (BioConductor) plus tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer,
    and signal from CRAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:ef4c7750830b38db61669171a5ed11720945ef341853cefefba3b1a7d8df1b4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# batch-normalization-comparison-across-methods

## Summary

Apply and compare the nine normalization methods implemented in subMALDI's normSpectra function (rel_TIC, custom, max, max_set, RMS, TIC, median, stdev, quantile) on a set of MALDI-MS replicate spectra held in a plain R data frame, verifying correctness via TIC invariants.

## When to use

Use when raw or pre-processed MALDI/ESI-MS replicate intensities sit in an R data frame (m/z column plus per-replicate intensity columns) and you must normalize them and/or compare which normalization scheme to adopt.

## When NOT to use

- Input intensities are already normalized (e.g. prior rel_TIC/TIC scaling applied) — re-normalizing will distort the relative scale
- Data are held as S3/S4 class objects from other MS packages (e.g. MALDIquant, MSnbase) rather than plain data frames — extract m/z/intensity columns into a data frame first, since subMALDI operates on unmasked data objects
- The task is peak detection or baseline correction rather than intensity scaling — use subMALDI's pre-processing functions instead

## Inputs

- Spectral data frame with all m/z values in the first column and each replicate's intensities in adjacent columns (irregularly-spaced MALDI-MS data)
- Sample spectra distributed with subMALDI (e.g. Before1, Before2)
- Choice of normalization method string: rel_TIC, custom, max, max_set, RMS, TIC, median, stdev, or quantile

## Outputs

- One normalized spectral data frame per method (nine total for a full comparison)
- rel_TIC verification result: per-spectrum TIC values, each exactly 1, with total set intensity equal to the number of spectra

## How to apply

Load the sample spectra shipped with subMALDI (e.g. Before1 and Before2) into a single spectral data frame where the first column holds all m/z values and each replicate's intensities occupy adjacent columns — subMALDI deliberately avoids S3/S4 objects so data stay accessible at every step. Call normSpectra once per method: 'rel_TIC' (with the total set equal to the number of spectra), plus 'custom', 'max', 'max_set', 'RMS', 'TIC', 'median', 'stdev', and 'quantile', yielding one normalized data frame per method. For 'rel_TIC', compute the column-sum TIC of every normalized spectrum and confirm each equals exactly 1; the summed intensities across the whole set should then equal the number of spectra (2 for the two-spectrum Before1/Before2 case). Rationale: normalization lets processed, irregularly-spaced replicate spectra be compared on a common intensity scale before averaging, subtraction, or plotting with subMALDI's function suite.

## Related tools

- **subMALDI** (Provides normSpectra and the nine normalization methods; the package's data-frame design keeps m/z and intensity data accessible after each normalization step) — https://github.com/wesleyburr/subMaldi
- **R** (Runtime environment in which subMALDI normalization is executed; requires MassSpecWavelet (BioConductor) plus tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, and signal from CRAN)

## Examples

```
normSpectra(spectra_df, method = "rel_TIC", total = 2); stopifnot(all(colSums(spectra_df_norm[, -1]) == 1))  # task_004: verify TIC == 1 per spectrum
```

## Evaluation signals

- After rel_TIC normalization, the column-sum TIC of every spectrum equals exactly 1 (floating-point-exact, not approximately)
- The sum of all intensities across the rel_TIC-normalized set equals the number of spectra (2 for Before1 + Before2)
- Exactly one normalized data frame is produced per requested method, all sharing the same first-column m/z grid
- Normalized output remains a plain data frame (no S3/S4 masking), so m/z and intensity columns are directly inspectable after each step
- All nine methods (custom, max, max_set, RMS, TIC, median, stdev, quantile, rel_TIC) run without error on the same input spectra

## Limitations

- subMALDI targets irregularly-spaced mass spectrometry data; spectra must be aligned to a common m/z grid at a specified resolution for replicate columns to be comparable
- No software version numbers for subMALDI, R, or dependencies are stated in the article, and the source install from the GitHub releases folder (e.g. subMALDI-1.0-2.tar.gz, type = "source") requires pre-installing MassSpecWavelet via BiocManager
- The 'custom' normalization method requires the user to supply and implement their own scaling function, which must be tested separately

## Evidence

- [intro] subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [abstract] The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns: "The resulting data frame contains all m/z data
in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [discussion] These replicates can then be normalized, averaged, subtracted, and plotted using subMALDI's suite of functions: "These replicates can then be normalized, averaged, subtracted, and plotted using `subMALDI`'s suite of functions"
- [readme] subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test: "As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
