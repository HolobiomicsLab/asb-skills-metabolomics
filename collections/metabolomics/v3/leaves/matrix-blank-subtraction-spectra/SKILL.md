---
name: matrix-blank-subtraction-spectra
description: Use when raw MALDI-MS or ESI-MS replicate spectra are loaded as R data
  frames (m/z in column 1, one intensity column per replicate) and matrix blank replicates
  (e.g. 'Blank1', 'Blank2') must be averaged and subtracted from sample columns (e.g.
  'Before1', 'Before2').
when_to_use_negative:
- Input spectra are irregularly spaced across replicates and have not yet been aligned
  to a common m/z grid at a specified resolution — alignment to the full m/z array
  must precede column-wise subtraction
- The goal is generic pre-processing such as smoothing, baseline correction, or peak
  detection rather than removing matrix background signal via blank subtraction
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: subMALDI
  role: R package providing avgSpectra (method = 'mean') to average matrix blank replicate
    columns and subSpectra (Blank_Var = 'Average') to subtract the averaged blank
    from sample intensity columns
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Execution environment in which the data-frame-based subtraction workflow is
    run and validated against the vignette output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:4ecbe56ac55ad4e5696bc750202d34b8e724e6c40a6ae1319e4766f7716f24b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# matrix-blank-subtraction-spectra

## Summary

A subMALDI-based procedure for removing matrix background signal from MALDI-MS replicate spectra: matrix blank replicates are averaged with avgSpectra(method = 'mean') and subtracted from sample intensity columns with subSpectra(Blank_Var = 'Average'), all on a plain data frame with m/z in the first column.

## When to use

Use when raw MALDI-MS or ESI-MS replicate spectra are loaded as R data frames (m/z in column 1, one intensity column per replicate) and matrix blank replicates (e.g. 'Blank1', 'Blank2') must be averaged and subtracted from sample columns (e.g. 'Before1', 'Before2').

## When NOT to use

- Input spectra are irregularly spaced across replicates and have not yet been aligned to a common m/z grid at a specified resolution — alignment to the full m/z array must precede column-wise subtraction
- The goal is generic pre-processing such as smoothing, baseline correction, or peak detection rather than removing matrix background signal via blank subtraction

## Inputs

- R data frame of matrix blank replicate spectra: m/z values in the first column, one intensity column per blank replicate (e.g. 'Blank1', 'Blank2')
- R data frame of sample replicate spectra: m/z values in the first column, one intensity column per sample replicate (e.g. 'Before1', 'Before2')
- subMALDI package installed from source (requires BiocManager + MassSpecWavelet, plus tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal)

## Outputs

- Data frame retaining m/z in the first column with new blank-subtracted intensity columns (e.g. 'Sub_Before1', 'Sub_Before2') adjacent to the originals
- Averaged blank intensity column stored under the 'Average' variable, referenced by Blank_Var in subSpectra

## How to apply

Load the bundled matrix blank replicate spectra and sample spectra into R as data frames, with all m/z values in the first column and each replicate's intensities in adjacent columns. Apply subMALDI's avgSpectra with method = 'mean' to combine the matrix blank replicate columns ('Blank1', 'Blank2') into a single averaged blank column stored under 'Average'. Then call subSpectra with Blank_Var = 'Average' on each sample column ('Before1', 'Before2'), which generates new blank-subtracted intensity columns named 'Sub_Before1' and 'Sub_Before2' in the same data frame. The rationale is that subMALDI works on plain data frames rather than S3/S4 objects, so m/z and intensity values stay accessible and inspectable after every processing step; validate by checking that the subtracted columns match the vignette's blank-subtracted intensities.

## Related tools

- **subMALDI** (R package providing avgSpectra (method = 'mean') to average matrix blank replicate columns and subSpectra (Blank_Var = 'Average') to subtract the averaged blank from sample intensity columns) — https://github.com/wesleyburr/subMaldi
- **R** (Execution environment in which the data-frame-based subtraction workflow is run and validated against the vignette output)

## Examples

```
blank <- avgSpectra(spectra, method = "mean", cols = c("Blank1", "Blank2")); spectra <- subSpectra(spectra, Blank_Var = "Average")
```

## Evaluation signals

- The output data frame retains m/z values in the first column with each sample's blank-subtracted intensities in adjacent columns
- New columns are named with the 'Sub_' prefix (e.g. 'Sub_Before1', 'Sub_Before2') corresponding to each input sample column
- An 'Average' column exists holding the mean of the matrix blank replicates, and subSpectra was invoked with Blank_Var = 'Average'
- Subtracted intensity values numerically match the blank-subtracted columns shown in the subMALDI vignette output for the Before1/Before2 replicates
- m/z and intensity values remain directly inspectable in the data frame after subtraction (no S3/S4 class masking)

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency, so exact reproducibility of vignette values may vary across versions
- No DOI or data availability statement is given for the sample workflow datasets used in the vignette
- The package must be installed from source (subMALDI-1.0-2.tar.gz) after the BioConductor dependency MassSpecWavelet, and the README notes the BioConductor install sequence does not always work cleanly
- Subtraction is column-wise and therefore presumes spectra were previously aligned to the same m/z array at a specified resolution

## Evidence

- [discussion] Discussion: "These replicates can then be normalized, averaged, subtracted, and plotted using `subMALDI`'s suite of functions"
- [abstract] Abstract: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [intro] Intro: "permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [readme] README: [evidence span withheld — anchor status: external_doc]
