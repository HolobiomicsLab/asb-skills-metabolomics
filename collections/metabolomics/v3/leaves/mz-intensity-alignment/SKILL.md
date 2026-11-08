---
name: mz-intensity-alignment
description: Use when replicate mass spectra have non-identical, irregularly-spaced
  m/z values and must be placed into one comparable data frame (m/z column plus per-replicate
  intensity columns) before averaging, normalization, or subtraction.
when_to_use_negative:
- The spectra are already on a common, regularly-spaced m/z axis, so cross-replicate
  comparison does not require grid mapping
- The workflow requires S3/S4 spectral object classes or external processing languages
  such as Python, Matlab, or C++ pipelines (MSnbase, pyOpenMS, LIMPIC, Proteo Wizard)
  rather than accessible R data frames
tools:
- name: subMALDI
  role: Provides createSpecDF() to build the theoretical m/z grid at a specified resolution,
    mapSpectrum() to place each replicate's intensities in adjacent columns, and rmveEmpty()
    to remove empty m/z bins
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the subMALDI alignment functions are executed;
    the package is installed from source after installing BiocManager/MassSpecWavelet
    and CRAN dependencies (tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:8004d963bedc5fa9a6f6c430c437ecad9ab74773f66dd039087ff6dd9ba2f3f3
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
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# mz-intensity-alignment

## Summary

Aligns irregularly-spaced MALDI/ESI mass spectra onto a shared grid of all theoretically possible m/z values at a specified instrument resolution, using subMALDI's createSpecDF(), mapSpectrum(), and rmveEmpty() in R to yield a single data frame comparable across spectral replicates.

## When to use

Use when replicate mass spectra have non-identical, irregularly-spaced m/z values and must be placed into one comparable data frame (m/z column plus per-replicate intensity columns) before averaging, normalization, or subtraction.

## When NOT to use

- The spectra are already on a common, regularly-spaced m/z axis, so cross-replicate comparison does not require grid mapping
- The workflow requires S3/S4 spectral object classes or external processing languages such as Python, Matlab, or C++ pipelines (MSnbase, pyOpenMS, LIMPIC, Proteo Wizard) rather than accessible R data frames

## Inputs

- Raw, irregularly-spaced mass spectral replicates (MALDI-MS or ESI-MS) as data-frame or file-backed spectra loadable in R
- Observed mass range (min and max m/z)
- Instrument resolution specifying the theoretical m/z grid spacing

## Outputs

- A mapped spectra data frame with all m/z values in the first column and each replicate's intensities in adjacent columns
- Row-reduced mapped data frame after rmveEmpty() removal of empty m/z bins (e.g. exported as mapped_spectra.csv)

## How to apply

In R, load subMALDI and first build the full theoretical m/z grid over the observed mass range with createSpecDF(), passing the instrument's specified resolution so every theoretically possible m/z value is represented. Next, map each raw replicate onto that grid with mapSpectrum(), which places each replicate's intensity values into columns adjacent to the shared first-column m/z values; repeat per replicate so every replicate becomes one adjacent column. Then apply rmveEmpty() to the mapped data frame to drop m/z bins where no replicate has measured intensity, reducing the row count. The rationale is that irregular spacing prevents direct replicate comparison, so alignment to a theoretical m/z array at the stated resolution is the prerequisite for downstream normalization, averaging, subtraction, and plotting with subMALDI's suite of functions.

## Related tools

- **subMALDI** (Provides createSpecDF() to build the theoretical m/z grid at a specified resolution, mapSpectrum() to place each replicate's intensities in adjacent columns, and rmveEmpty() to remove empty m/z bins) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the subMALDI alignment functions are executed; the package is installed from source after installing BiocManager/MassSpecWavelet and CRAN dependencies (tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal))

## Examples

```
specDF <- createSpecDF(min_mz, max_mz, resolution); mapped <- mapSpectrum(specDF, raw_spectrum); mapped <- rmveEmpty(mapped); write.csv(mapped, "mapped_spectra.csv")
```

## Evaluation signals

- Structural check: the output data frame has all m/z data in the first column and each spectral replicate's intensity data in adjacent columns
- Row-count check: rmveEmpty() strictly decreased the number of rows relative to the pre-filter mapped data frame (record row counts before and after)
- Grid completeness: the first column contains all theoretically possible m/z values in the observed mass range at the specified resolution
- Replicate completeness: the number of adjacent intensity columns equals the number of input spectral replicates
- Persistence check: the mapped data frame was saved as mapped_spectra.csv without loss of m/z or intensity accessibility (no S3/S4 masking)

## Limitations

- No software version numbers are stated for subMALDI, R, or dependencies, so exact reproducibility of grid spacing is not version-pinned
- No DOI or data availability statement is given for the datasets used in the workflow
- The package must be installed from source (e.g. subMALDI-1.0-2.tar.gz with type = "source"), and the BioConductor dependency MassSpecWavelet can be problematic to install outside the documented BiocManager sequence

## Evidence

- [abstract] aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution: "aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution"
- [abstract] The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [abstract] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [readme] install.packages("subMALDI-1.0-2.tar.gz", type = "source"): "install.packages("subMALDI-1.0-2.tar.gz", type = "source")"
