---
name: spectral-gridded-mapping
description: Use when replicates of MALDI-MS or ESI-MS spectra have irregularly spaced
  m/z values and must be standardized onto a common theoretical m/z grid so their
  intensities can be compared, averaged, or normalized.
when_to_use_negative:
- The replicate spectra are already aligned to a common m/z axis — re-gridding is
  unnecessary and rmveEmpty() row-reduction cannot be demonstrated
- The workflow requires S3/S4 class-based spectral objects or the processing pipelines
  of packages like MALDIquant or MSnbase rather than plain data-frame objects
- The data are not mass spectrometry intensity-vs-m/z measurements, so no theoretical
  m/z grid can be defined
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: subMALDI
  role: Supplies createSpecDF(), mapSpectrum(), and rmveEmpty(), the three functions
    that build the theoretical m/z grid, map each replicate's intensities onto it,
    and remove empty rows
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the subMALDI mapping workflow is run; install
    subMALDI from source alongside Bioconductor's MassSpecWavelet and CRAN packages
    (tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal)
  repo: https://github.com/wesleyburr/subMaldi
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:5dae6c60dd5e641fd504c734266f61eec43b8ce28b6102aad5d63f730d8c061d
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
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-gridded-mapping

## Summary

Maps irregularly spaced mass spectral replicates (MALDI-MS/ESI-MS) onto an array of all theoretically possible m/z values at a specified resolution using the subMALDI R package, producing a single data frame with a shared m/z column and one intensity column per replicate.

## When to use

Use when replicates of MALDI-MS or ESI-MS spectra have irregularly spaced m/z values and must be standardized onto a common theoretical m/z grid so their intensities can be compared, averaged, or normalized.

## When NOT to use

- The replicate spectra are already aligned to a common m/z axis — re-gridding is unnecessary and rmveEmpty() row-reduction cannot be demonstrated
- The workflow requires S3/S4 class-based spectral objects or the processing pipelines of packages like MALDIquant or MSnbase rather than plain data-frame objects
- The data are not mass spectrometry intensity-vs-m/z measurements, so no theoretical m/z grid can be defined

## Inputs

- Raw or pre-processed mass spectral replicate files (e.g. the sample spectra bundled with the subMALDI package) with irregularly spaced m/z points
- Instrument resolution (specifies the density of the theoretical m/z grid)
- Observed mass range (min/max m/z for createSpecDF)

## Outputs

- A data frame whose first column contains all m/z values and whose adjacent columns contain each replicate's intensity data
- Row-reduced version of that data frame after rmveEmpty() removes empty m/z bins
- Mapped spectra exported as a CSV (e.g. mapped_spectra.csv)

## How to apply

Build the full theoretical m/z grid over the observed mass range with subMALDI's createSpecDF(), specifying the instrument resolution so the grid contains every theoretically possible m/z value. Map each replicate onto that grid with mapSpectrum(), which places each replicate's intensity values into columns adjacent to the shared first-column m/z values. Apply rmveEmpty() to drop m/z bins where no replicate has a measured intensity, reducing the number of rows. Inspect the result to confirm the first column holds all m/z values and adjacent columns hold each replicate's intensities, record row counts before and after rmveEmpty(), and save the mapped frame (e.g. mapped_spectra.csv). The rationale, per the article, is that subMALDI keeps m/z and intensity data accessible in plain data frames rather than masking them into S3 or S4 class objects, so the gridded output stays compatible with functions from other R packages.

## Related tools

- **subMALDI** (Supplies createSpecDF(), mapSpectrum(), and rmveEmpty(), the three functions that build the theoretical m/z grid, map each replicate's intensities onto it, and remove empty rows) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the subMALDI mapping workflow is run; install subMALDI from source alongside Bioconductor's MassSpecWavelet and CRAN packages (tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal)) — https://github.com/wesleyburr/subMaldi

## Examples

```
install.packages("BiocManager"); BiocManager::install("MassSpecWavelet"); install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE)
```

## Evaluation signals

- Structural criterion: the output data frame's first column contains all m/z values, with each spectral replicate's intensity data in adjacent columns
- rmveEmpty() must strictly decrease the row count relative to the fully gridded data frame (record rows before and after)
- The m/z grid covers the full observed mass range at the specified instrument resolution (all theoretically possible m/z values present)
- Every replicate contributes one intensity column to the final data frame; mapped frame saves cleanly as mapped_spectra.csv

## Limitations

- Installing the BioConductor dependency MassSpecWavelet before subMALDI can fail; the README notes 'the BioConductor sequence doesn't always seem to work' and recommends installing the source tarball with dependencies = TRUE as a fallback
- No software version numbers, changelog, or DOI/data-availability statement are provided for the workflow datasets, so reproducibility of exact grid parameters depends on the user's stated resolution
- The full theoretical m/z grid can contain many empty bins; rmveEmpty() is required to trim them, and skipping it leaves a sparse data frame

## Evidence

- [abstract] abstract: "aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
