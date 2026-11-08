---
name: irregularly-spaced-spectral-data-handling
description: Use when raw mass spectrometry spectra (MALDI-MS or ESI-MS) are irregularly
  spaced on the m/z axis and you need smoothing, baseline correction, peak detection,
  normalization, or replicate alignment while keeping m/z and intensity values accessible
  as plain data frames.
when_to_use_negative:
- Your data are already managed in S3/S4 class objects from other R packages such
  as MSnbase or MALDIquant and you intend to remain within those ecosystems rather
  than unmask to plain data frames
- The input is not mass spectrometry spectral data (subMALDI targets organization,
  pre-processing, and normalization of spectral data sets, not generic signal tables)
- 'You need version-pinned, changelog-tracked tooling: no software version numbers
  beyond the tarball name, no changelog, and no DOI/data availability statement for
  the workflow datasets are provided'
edam_operation: http://edamontology.org/operation_3219
tools:
- name: subMALDI
  role: Core R package providing the data structure (plain data frames of m/z and
    intensity) and the organization, smoothing, baseline correction, peak detection,
    and normalization functions for irregularly-spaced spectra
  repo: https://github.com/wesleyburr/subMaldi
- name: MassSpecWavelet
  role: BioConductor dependency whose CWT algorithm (cwt, getLocalMaximumCWT, getRidge,
    identifyMajorPeaks with SNR.Th = 3) can be applied directly to subMALDI data frames
    for baseline correction and peak detection
- name: R
  role: Programming environment in which subMALDI runs and through which external-package
    compatibility (e.g. MassSpecWavelet) is exercised on subMALDI data objects
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:3b8efdac3e7cc93efa2167d8dbc7e18b30a8c4f41edfbca42dde17d2fb65d114
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3219
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# irregularly-spaced-spectral-data-handling

## Summary

Handles MALDI-MS/ESI-MS spectra that are irregularly spaced on the m/z axis by organizing, pre-processing (smoothing, baseline correction, peak detection), and normalizing them with the subMALDI R package, which keeps data in plain data frames rather than S3/S4 class objects.

## When to use

Use when raw mass spectrometry spectra (MALDI-MS or ESI-MS) are irregularly spaced on the m/z axis and you need smoothing, baseline correction, peak detection, normalization, or replicate alignment while keeping m/z and intensity values accessible as plain data frames.

## When NOT to use

- Your data are already managed in S3/S4 class objects from other R packages such as MSnbase or MALDIquant and you intend to remain within those ecosystems rather than unmask to plain data frames
- The input is not mass spectrometry spectral data (subMALDI targets organization, pre-processing, and normalization of spectral data sets, not generic signal tables)
- You need version-pinned, changelog-tracked tooling: no software version numbers beyond the tarball name, no changelog, and no DOI/data availability statement for the workflow datasets are provided

## Inputs

- Raw MALDI-MS or ESI-MS spectra as m/z and intensity value pairs, irregularly spaced on the m/z axis (e.g. the bundled bsline raw spectrum loaded via readSpec)
- subMALDI package source tarball (e.g. subMALDI-1.0-2.tar.gz) installed from the GitHub releases folder with type = "source"

## Outputs

- Data frames with all m/z data in the first column and each spectral replicate's intensity data in adjacent columns
- Pre-processed and normalized spectra (smoothed, baseline-corrected, peak-detected)
- Peak lists as plain tables (e.g. peaks_bsline_cwt.csv with m/z and intensity columns filtered by an SNR criterion)

## How to apply

Load raw spectra with subMALDI's readSpec into data frames of m/z and intensity values, deliberately avoiding S3/S4 class masking so every processing step leaves m/z and intensity data directly accessible. Pre-process via the package's smoothing filters, baseline correction methods, and peak detection algorithms, and normalize or average/subtract/plot replicate spectra using subMALDI's function suite. When replicates must be compared, align processed intensity values to an array of all theoretically possible m/z values in the observed mass range at a specified resolution, producing a data frame with all m/z values in the first column and each replicate's intensities in adjacent columns. Because subMALDI data objects stay as data frames, external packages like the BioConductor MassSpecWavelet can be applied directly (e.g. cwt over scales 1-40, getLocalMaximumCWT, getRidge, identifyMajorPeaks with SNR.Th = 3) for baseline correction and peak detection in one pipeline.

## Related tools

- **subMALDI** (Core R package providing the data structure (plain data frames of m/z and intensity) and the organization, smoothing, baseline correction, peak detection, and normalization functions for irregularly-spaced spectra) — https://github.com/wesleyburr/subMaldi
- **MassSpecWavelet** (BioConductor dependency whose CWT algorithm (cwt, getLocalMaximumCWT, getRidge, identifyMajorPeaks with SNR.Th = 3) can be applied directly to subMALDI data frames for baseline correction and peak detection)
- **R** (Programming environment in which subMALDI runs and through which external-package compatibility (e.g. MassSpecWavelet) is exercised on subMALDI data objects)

## Examples

```
BiocManager::install("MassSpecWavelet"); install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE)
```

## Evaluation signals

- Output remains a plain data frame (no S3/S4 masking), with m/z values in the first column and each replicate's intensities in adjacent columns
- Aligned spectra cover all theoretically possible m/z values in the observed mass range at the specified resolution
- After SNR-based peak filtering (SNR.Th = 3 with flag-and-remove), the output peak table's row count is reduced relative to the pre-filter peak list, retaining only peaks above the threshold
- Replicate spectra can be normalized, averaged, subtracted, and plotted using subMALDI's function suite without data leaving data-frame form
- External package functions (e.g. MassSpecWavelet CWT) run directly on subMALDI data structures without format conversion

## Limitations

- No software version numbers are stated for subMALDI, R, or any dependency; only the tarball name subMALDI-1.0-2.tar.gz implies a version
- No changelog/version history is provided for the package, and no citation or DOI is given for the subMALDI package itself within the repo README beyond the JOSS badge
- The README warns that the BioConductor install sequence (BiocManager + MassSpecWavelet) does not always work as given; installing the tarball with dependencies = TRUE is offered as the workaround
- No DOI or data availability statement is provided for the datasets used in the workflow

## Evidence

- [abstract] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [abstract] The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns: "The resulting data frame contains all m/z data
in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [intro] subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test: "As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] subMALDI is an open framework package for the R programming environment that permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [discussion] These replicates can then be normalized, averaged, subtracted, and plotted using `subMALDI`'s suite of functions: "These replicates can then be normalized, averaged, subtracted, and plotted using `subMALDI`'s suite of functions"
