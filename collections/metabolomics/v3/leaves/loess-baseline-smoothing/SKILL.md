---
name: loess-baseline-smoothing
description: Use when MALDI-MS or ESI-MS spectra carry an irregular, drifting baseline
  that must be estimated with LOESS and subtracted before peak detection or replicate
  comparison, and the data must remain accessible as plain data frames.
when_to_use_negative:
- The input spectrum already has a near-zero flat baseline — subtracting another estimated
  baseline would distort peak intensities
- The workflow requires Bioconductor S4 container classes (e.g. MALDIquant/MSnbase-style
  objects) — subMALDI intentionally keeps data as plain data frames instead
- The goal is the monotone_min + linear interpolation (n=7) baseline route demonstrated
  in the vignette rather than the LOESS method
edam_operation: http://edamontology.org/operation_3215
tools:
- name: subMALDI
  role: Provides the LOESS baseline estimator used to correct the spectra, as one
    of three baselineCorr() methods
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which subMALDI is installed from source and the
    baseline correction is run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:aed7cfd9edb680f93c54fce0b7811ada0122761499b418b72291f377e2ee06bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3215
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# loess-baseline-smoothing

## Summary

Estimates and removes the drifting baseline of MALDI/ESI mass spectra using the subMALDI R package's LOESS (local polynomial regression) baseline estimator, applied to spectra held as plain m/z/intensity data frames rather than S3/S4 class objects.

## When to use

Use when MALDI-MS or ESI-MS spectra carry an irregular, drifting baseline that must be estimated with LOESS and subtracted before peak detection or replicate comparison, and the data must remain accessible as plain data frames.

## When NOT to use

- The input spectrum already has a near-zero flat baseline — subtracting another estimated baseline would distort peak intensities
- The workflow requires Bioconductor S4 container classes (e.g. MALDIquant/MSnbase-style objects) — subMALDI intentionally keeps data as plain data frames instead
- The goal is the monotone_min + linear interpolation (n=7) baseline route demonstrated in the vignette rather than the LOESS method

## Inputs

- Raw or smoothed MALDI/ESI mass spectra as two-column plain data frames (m/z column, intensity column)
- bsline/smoothed example spectra files bundled with the subMALDI package

## Outputs

- Baseline-corrected spectrum (intensities with the LOESS-estimated baseline subtracted)
- Individual text file per spectrum with m/z and corrected intensity columns, matching the vignette figure layout

## How to apply

Load the bsline/smoothed example spectra bundled with subMALDI (github.com/wesleyburr/subMaldi) into R as plain data-frame objects with m/z in one column and intensity in another — the package deliberately avoids masking data into S3/S4 objects so m/z and intensity stay accessible after each step. Install subMALDI from source (subMALDI-1.0-2.tar.gz, type = "source", dependencies = TRUE) after installing the BioConductor dependency MassSpecWavelet and CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, and signal. Apply subMALDI's LOESS (local polynomial regression smoothing) baseline estimator to the spectra as one of the three baselineCorr() methods demonstrated in the vignette — alongside monotone_min and linear interpolation (n=7) — and subtract the fitted baseline from the raw intensity trace. Choose LOESS over the monotone-minimum/linear-interpolation route when a local-regression smooth of the baseline is preferred; the vignette runs all three methods so their corrected outputs can be compared. Write each baseline-corrected spectrum to an individual text file with m/z and corrected intensity columns, matching the file layout of the vignette figures.

## Related tools

- **subMALDI** (Provides the LOESS baseline estimator used to correct the spectra, as one of three baselineCorr() methods) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which subMALDI is installed from source and the baseline correction is run)

## Evaluation signals

- The corrected spectrum's baseline runs along the x-axis at a near-zero flat level, as shown for all three methods in the vignette
- Peak intensities are retained after baseline subtraction — correction removes the irregular baseline without flattening peaks
- Output text file contains exactly two columns (m/z and corrected intensity), matching the file layout the vignette figures depict
- Data remain a plain data frame after correction (no S3/S4 masking), so the m/z column is still directly accessible
- The LOESS-corrected result can be compared side-by-side against the monotone_min and linear (n=7) corrected spectra on the same input

## Limitations

- subMALDI must be installed from source; the README warns the BioConductor MassSpecWavelet dependency sequence 'doesn't always seem to work' and recommends install.packages(..., type = "source", dependencies = TRUE) instead
- No software version numbers or changelog are stated for subMALDI, R, or dependencies, so reproducibility of the baseline-correction parameters is limited
- The abstract does not name the specific baseline correction methods or their parameters — the vignette (subMALDIprocessing) must be consulted for the LOESS demonstration

## Evidence

- [abstract] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [intro] subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test.: "As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] begin by downloading the source copy of the package from the releases folder and install it using type = "source".: "begin by downloading the source copy of the package from the [releases folder](https://github.com/wesleyburr/subMaldi/releases) and 
install it using type = "source"."
- [readme] if you install from source, and set dependencies = TRUE, this should all be moot.: [evidence span withheld — anchor status: external_doc]
