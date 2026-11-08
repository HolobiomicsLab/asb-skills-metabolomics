---
name: mass-spectrometry-data-frame-construction
description: Use when you have multiple raw spectral replicates with irregularly spaced
  m/z values that must be placed on a shared, regularly spaced m/z grid before normalization,
  averaging, or replicate-wise comparison.
when_to_use_negative:
- The spectra are already aligned on a common m/z grid and only downstream pre-processing
  (smoothing, baseline correction, peak detection, normalization) is needed
- The data already live in S3/S4 spectral objects from packages such as MALDIquant
  or MSnbase and you intend to stay within those ecosystems' own alignment routines
- Only a single spectrum needs analysis with no replicate comparison — grid alignment
  exists to make replicates comparable
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: subMALDI
  role: Provides createSpecDF(), mapSpectrum(), and rmveEmpty() that construct the
    m/z grid, map replicate intensities onto it, and prune empty bins
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the package is installed from source and
    the mapping workflow is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:b0008e02d10efdd74cd861c9bb226120c4b49efb3214b5b34e6149e6c30f4c44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mass-spectrometry-data-frame-construction

## Summary

Maps irregularly-spaced MALDI-MS/ESI-MS replicate spectra into a single rectangular data frame by aligning each spectrum to an array of all theoretically possible m/z values at a specified instrument resolution, so replicates become directly comparable columns.

## When to use

Use when you have multiple raw spectral replicates with irregularly spaced m/z values that must be placed on a shared, regularly spaced m/z grid before normalization, averaging, or replicate-wise comparison.

## When NOT to use

- The spectra are already aligned on a common m/z grid and only downstream pre-processing (smoothing, baseline correction, peak detection, normalization) is needed
- The data already live in S3/S4 spectral objects from packages such as MALDIquant or MSnbase and you intend to stay within those ecosystems' own alignment routines
- Only a single spectrum needs analysis with no replicate comparison — grid alignment exists to make replicates comparable

## Inputs

- Raw replicate spectra files (MALDI-MS or ESI-MS, irregularly spaced m/z values) shipped or supplied as loadable spectra
- Instrument resolution used to generate the theoretical m/z grid
- Observed mass range (min/max m/z) for grid construction

## Outputs

- A data frame whose first column contains all m/z values, with each spectral replicate's intensity data in adjacent columns
- Row-count record before and after rmveEmpty() demonstrating removal of empty m/z bins
- Exported table (e.g. mapped_spectra.csv)

## How to apply

In R with subMALDI, first build the full theoretical m/z grid over the observed mass range using createSpecDF(), passing the instrument resolution so the grid contains every theoretically possible m/z value. Then run mapSpectrum() on each raw replicate to place its intensity values into columns adjacent to the shared first-column m/z values of the data frame. Finally apply rmveEmpty() to drop m/z bins where no replicate has measured intensity, reducing the row count. Record row counts before and after rmveEmpty() to confirm empty-bin removal, and export the result (e.g. mapped_spectra.csv). The design deliberately avoids masking data into S3/S4 class objects so m/z and intensity values remain accessible and compatible with functions from other R packages at every step.

## Related tools

- **subMALDI** (Provides createSpecDF(), mapSpectrum(), and rmveEmpty() that construct the m/z grid, map replicate intensities onto it, and prune empty bins) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the package is installed from source and the mapping workflow is executed)

## Examples

```
BiocManager::install("MassSpecWavelet"); install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE)
```

## Evaluation signals

- Structural check: the output data frame's first column contains all m/z values and each replicate's intensities occupy adjacent columns (the paper's stated structural criterion)
- The m/z grid spans the full observed mass range and was generated with the specified instrument resolution (all theoretically possible m/z values present)
- row count of the data frame strictly decreases after rmveEmpty(), confirming empty m/z bins were removed
- m/z and intensity values remain directly accessible in plain data-frame columns (no S3/S4 object masking) after mapping
- Exported mapped_spectra.csv round-trips: all replicate columns align row-wise to the single shared m/z column

## Limitations

- No software version numbers are stated for subMALDI, R, or its dependencies, so reproducibility of exact grid spacing must be pinned by the user
- No DOI or data availability statement is given for the datasets used in the published workflow
- The package must be installed from source (with the BioConductor dependency MassSpecWavelet installed first); the BioConductor installation sequence 'doesn't always seem to work' per the README
- The article provides no changelog or citation/DOI for the package release itself

## Evidence

- [abstract] grid construction and mapping: "aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution"
- [abstract] output data-frame structure: "The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [readme] open, unmasked data objects: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
