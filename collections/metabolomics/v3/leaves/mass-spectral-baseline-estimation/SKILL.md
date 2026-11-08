---
name: mass-spectral-baseline-estimation
description: Use when raw MALDI-MS or ESI-MS intensity traces show an irregular, drifting
  baseline that must be removed before peak detection or normalization, and m/z/intensity
  data are held as plain data frames.
when_to_use_negative:
- The input is already baseline-corrected and only needs smoothing, peak detection,
  or normalization
- Spectra are stored in S3/S4 class objects from other packages (e.g., MALDIquant,
  MSnbase) and cannot be provided as plain m/z/intensity data frames
- The analysis goal is alignment of spectra to an m/z grid rather than baseline removal
tools:
- name: subMALDI
  role: Provides the baselineCorr() function with monotone_min, linear, and loess
    baseline estimation methods applied to m/z/intensity data frames
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Runtime environment in which subMALDI baselineCorr() is called on spectra
    loaded as plain data-frame objects
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:0c09a27c029d87d78fcf9a2d647ec0ee73be961e0326fbb8c0712f6a4d1db82e
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
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectral-baseline-estimation

## Summary

Estimates and subtracts the underlying baseline of MALDI mass spectra using the subMALDI package's baselineCorr() with one of three methods — monotone_min, linear, or loess — producing baseline-corrected spectra whose baseline runs along the x-axis while peak intensities are retained.

## When to use

Use when raw MALDI-MS or ESI-MS intensity traces show an irregular, drifting baseline that must be removed before peak detection or normalization, and m/z/intensity data are held as plain data frames.

## When NOT to use

- The input is already baseline-corrected and only needs smoothing, peak detection, or normalization
- Spectra are stored in S3/S4 class objects from other packages (e.g., MALDIquant, MSnbase) and cannot be provided as plain m/z/intensity data frames
- The analysis goal is alignment of spectra to an m/z grid rather than baseline removal

## Inputs

- Raw mass spectrum as an R data frame with m/z values in one column and intensity values in another (e.g., the subMALDI bsline/smoothed example spectra)
- Irregularly-spaced MALDI-MS replicate spectra bundled with the subMALDI package

## Outputs

- Baseline-corrected spectrum with m/z and corrected intensity columns
- Per-spectrum text file with m/z and corrected intensity columns matching the vignette figure layout

## How to apply

Load the example bsline/smoothed spectra (m/z in one column, intensity in another) from github.com/wesleyburr/subMaldi into R as plain data-frame objects — subMALDI deliberately avoids S3/S4 masking so m/z and intensity stay accessible after each processing step. Apply baselineCorr() choosing among the three estimation methods: monotone_min (monotone minimum followed by linear interpolation to estimate the underlying baseline), linear (used in the vignette with n=7), or loess (local polynomial regression smoothing) as the alternative estimator. Subtract the fitted baseline from the raw intensity trace for each spectrum. Decision point: monotone_min is the vignette's monotone-minimum + linear-interpolation route; loess is the smoother alternative when the minimum-based estimate is too rough. Write each corrected spectrum to a text file with m/z and corrected intensity columns, matching the layout of the vignette figures.

## Related tools

- **subMALDI** (Provides the baselineCorr() function with monotone_min, linear, and loess baseline estimation methods applied to m/z/intensity data frames) — https://github.com/wesleyburr/subMaldi
- **R** (Runtime environment in which subMALDI baselineCorr() is called on spectra loaded as plain data-frame objects)

## Examples

```
corrected <- subMALDI::baselineCorr(spectrum, method = "linear", n = 7)
```

## Evaluation signals

- Each corrected spectrum's baseline runs along the x-axis, matching the subMALDI vignette figures
- Corrected spectra show a near-zero flat baseline while retaining peak intensities
- The raw spectrum's irregularities are visibly removed relative to the raw trace in comparison plots
- Output text files contain exactly two columns (m/z and corrected intensity), matching the vignette file layout
- All three methods (monotone_min, linear with n=7, loess) reproduce the vignette's corrected-spectra panels when run on the bsline/smoothed examples

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency, so exact method behaviour may vary across versions
- The article names baseline correction methods generically and gives no method-specific thresholds beyond the vignette's linear n=7 parameter
- No DOI or data availability statement is given for the datasets used in the workflow

## Evidence

- [intro] subMALDI is an open framework for baseline correction of irregularly-spaced spectra: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [abstract] baseline correction as package workflow step: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [full_text] vignette demonstrates all three baselineCorr methods: [evidence span withheld — anchor status: not_found]
- [readme] install package from source with dependencies: [evidence span withheld — anchor status: external_doc]
