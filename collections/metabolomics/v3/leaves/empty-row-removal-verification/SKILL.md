---
name: empty-row-removal-verification
description: Use when irregularly-spaced MALDI/ESI spectral replicates have been mapped
  onto a theoretical m/z grid with subMALDI (createSpecDF/mapSpectrum) and the row
  count must be reduced and validated before saving the mapped data frame.
when_to_use_negative:
- Spectra that are already aligned to a shared m/z grid with no empty bins — rmveEmpty()
  would have nothing to remove and its row-reduction validation would be meaningless
- Workflows using MSnbase, MALDIquant, or pyOpenMS that wrap spectra in S3/S4 class
  objects — subMALDI's data-frame-based rmveEmpty() is designed to avoid such object
  masking
- Peak lists or feature tables extracted after peak detection — this skill operates
  on raw mapped intensity matrices, not detected-peak output
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: subMALDI
  role: Source of rmveEmpty() (row reduction of empty m/z bins), together with createSpecDF()
    and mapSpectrum() that build the theoretical m/z grid and place replicate intensities
    into adjacent columns
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Environment in which the subMALDI data frame workflow (grid creation, mapping,
    empty-row removal, row-count verification) is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:62d7c871051eca1140b026b3d6b5a428683e334f7d87475ed7bfe0e63a088b01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# empty-row-removal-verification

## Summary

Verifies that subMALDI's rmveEmpty() step, applied after mapping spectra onto the theoretical m/z grid, removes rows for m/z bins where no replicate has measured intensity and confirms the mapped data frame retains the required structure (m/z in the first column, replicate intensities in adjacent columns).

## When to use

Use when irregularly-spaced MALDI/ESI spectral replicates have been mapped onto a theoretical m/z grid with subMALDI (createSpecDF/mapSpectrum) and the row count must be reduced and validated before saving the mapped data frame.

## When NOT to use

- Spectra that are already aligned to a shared m/z grid with no empty bins — rmveEmpty() would have nothing to remove and its row-reduction validation would be meaningless
- Workflows using MSnbase, MALDIquant, or pyOpenMS that wrap spectra in S3/S4 class objects — subMALDI's data-frame-based rmveEmpty() is designed to avoid such object masking
- Peak lists or feature tables extracted after peak detection — this skill operates on raw mapped intensity matrices, not detected-peak output

## Inputs

- Mapped spectral data frame produced by createSpecDF() + mapSpectrum(): first column of all theoretical m/z values, adjacent columns of replicate intensities
- Instrument resolution used to generate the theoretical m/z grid
- Bundled sample spectra files shipped with the subMALDI R package

## Outputs

- Row-reduced mapped data frame with empty m/z bins removed (saved as mapped_spectra.csv)
- Row counts before and after rmveEmpty() demonstrating strict row reduction
- Structural confirmation that column 1 is m/z and adjacent columns are replicate intensities

## How to apply

After building the full theoretical m/z grid with createSpecDF() (passing the instrument resolution so all theoretically possible m/z values in the observed mass range appear) and mapping each replicate with mapSpectrum(), record the number of rows of the mapped data frame before pruning. Apply rmveEmpty() to the mapped data frame so that m/z bins for which no replicate has measured intensity are dropped, then re-count rows to confirm the count strictly decreased. Inspect the pruned data frame to confirm the structural criterion from the paper: all m/z values in the first column and each replicate's intensity values in adjacent columns. Record pre- and post-pruning row counts as evidence and save the result as mapped_spectra.csv for downstream pre-processing (smoothing, baseline correction, peak detection, normalization).

## Related tools

- **subMALDI** (Source of rmveEmpty() (row reduction of empty m/z bins), together with createSpecDF() and mapSpectrum() that build the theoretical m/z grid and place replicate intensities into adjacent columns) — https://github.com/wesleyburr/subMaldi
- **R** (Environment in which the subMALDI data frame workflow (grid creation, mapping, empty-row removal, row-count verification) is executed)

## Examples

```
rows_before <- nrow(mapped_df); mapped_df <- rmveEmpty(mapped_df); stopifnot(nrow(mapped_df) < rows_before); write.csv(mapped_df, "mapped_spectra.csv")
```

## Evaluation signals

- Number of rows strictly decreases after rmveEmpty() — record counts before and after to prove empty-bin removal occurred
- Structural invariant: the output data frame's first column contains all m/z values and each replicate's intensity data sits in adjacent columns
- No retained m/z bin has zero measured intensity across all replicates
- Output saved as mapped_spectra.csv is loadable and compatible with downstream subMALDI steps (smoothing, baseline correction, peak detection, normalization)
- All subMALDI objects remain plain data frames (no S3/S4 masking), so m/z and intensity columns stay directly accessible after pruning

## Limitations

- No software version numbers are stated for subMALDI, R, or dependencies, so the exact rmveEmpty() behavior cannot be pinned to a release
- subMALDI installs from source (subMALDI-1.0-2.tar.gz) and requires the BioConductor dependency MassSpecWavelet; the BioConductor install sequence 'doesn't always seem to work' per the README
- Row reduction depends on the resolution chosen for createSpecDF(): coarser grids leave fewer empty bins, so verification results are resolution-dependent
- No changelog/version history is provided for the package

## Evidence

- [abstract] The resulting data frame contains all m/z data in the first column, with the intensity data of each spectral replicate in adjacent columns: "The resulting data frame contains all m/z data
in the first column, with the intensity data of each spectral replicate in adjacent columns"
- [abstract] aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution: "aligned to an array of all the theoretically possible m/z values in the
observed mass range, at a specified resolution"
- [readme] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets.: "permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets."
- [readme] install.packages("subMALDI-1.0-2.tar.gz", type = "source", dependencies = TRUE): [evidence span withheld — anchor status: external_doc]
