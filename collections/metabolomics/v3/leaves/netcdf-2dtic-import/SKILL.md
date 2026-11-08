---
name: netcdf-2dtic-import
description: Use when raw comprehensive two-dimensional gas chromatography–mass spectrometry
  (GC×GC-MS) data is stored in NetCDF format and must be converted into a 2D-TIC matrix
  for smoothing, baseline correction, peak alignment, or MPCA in R.
when_to_use_negative:
- Data is already a processed chromatogram object (e.g. baseline-corrected or smoothed)
  — this import step is only for raw NetCDF files
- Input is 1D chromatography data (conventional GC-MS) where the two-dimensional folding
  step does not apply
- Data is not in NetCDF format and cannot be converted — RGCxGC imports the raw chromatogram
  specifically from a NetCDF file
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RGCxGC
  role: R package providing read_chrom to import the raw NetCDF chromatogram and fold
    it into the 2D-TIC
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Backend dependency for reading the raw chromatogram from the NetCDF file
- name: R
  role: Runtime environment in which the RGCxGC import workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:74fa0b6d16949b95ef802a8d9df348e1b0675afd1bcad25bf8947f71da60bd19
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  repo_url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - R
  - RGCxGC
  - RNetCDF
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# netcdf-2dtic-import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import raw GC×GC-MS chromatograms from NetCDF files and fold them into a two-dimensional Total Intensity Chromatogram (2D-TIC) using the RGCxGC package's read_chrom function, the mandatory first step of the RGCxGC preprocessing and multivariate analysis workflow.

## When to use

Use when raw comprehensive two-dimensional gas chromatography–mass spectrometry (GC×GC-MS) data is stored in NetCDF format and must be converted into a 2D-TIC matrix for smoothing, baseline correction, peak alignment, or MPCA in R.

## When NOT to use

- Data is already a processed chromatogram object (e.g. baseline-corrected or smoothed) — this import step is only for raw NetCDF files
- Input is 1D chromatography data (conventional GC-MS) where the two-dimensional folding step does not apply
- Data is not in NetCDF format and cannot be converted — RGCxGC imports the raw chromatogram specifically from a NetCDF file

## Inputs

- Raw chromatogram in NetCDF file format (GC×GC-MS output, e.g. from the MTBLS579 MetaboLights dataset)
- Modulation time and sampling rate metadata for the instrument run

## Outputs

- Two-dimensional Total Intensity Chromatogram (2D-TIC) R object suitable for baseline_corr, wsmooth, and batch_2DCOW in the RGCxGC package

## How to apply

Load the RGCxGC package in R and call read_chrom on the raw NetCDF chromatogram file, supplying the modulation time (mod_time), sampling rate (sam_rate), and per-evaluation parameters so the one-dimensional signal is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC). The mod_time parameter defines the second-dimension period used to slice the raw signal into rows of the 2D matrix, while x_cut and y_cut can trim retention-time ranges. The resulting chromatogram object becomes the input to baseline_corr, wsmooth, and downstream 2D-COW alignment or m_prcomp MPCA. Correctness hinges on a correctly specified modulation time: an incorrect value misaligns the fold and distorts every subsequent preprocessing step.

## Related tools

- **RGCxGC** (R package providing read_chrom to import the raw NetCDF chromatogram and fold it into the 2D-TIC) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Backend dependency for reading the raw chromatogram from the NetCDF file)
- **R** (Runtime environment in which the RGCxGC import workflow is executed)

## Examples

```
library(RGCxGC); MTBLS08_raw <- read_chrom(name = "08GB.cdf", mod_time = 3, sam_rate = 200)
```

## Evaluation signals

- The imported object is a valid 2D-TIC with dimensions consistent with the modulation time and total acquisition time (first dimension × second dimension grid)
- Plotting the imported chromatogram (e.g. via the filled.contour-based plotting in RGCxGC) shows a coherent 2D contour rather than a folded/garbled signal
- The 2D-TIC can be passed to baseline_corr or wsmooth without dimension or sampling-rate errors
- For benchmark data, the import reproduces the vignette's 08GB chromatogram from the MTBLS579 dataset (identifier https://www.ebi.ac.uk/metabolights/MTBLS579)

## Limitations

- The raw file must be in NetCDF format; other vendor formats require prior conversion
- A wrong mod_time or sam_rate silently folds the signal incorrectly, corrupting all downstream smoothing, baseline correction, and alignment
- Trimming via x_cut/y_cut may be needed to remove non-chemical regions before multivariate analysis

## Evidence

    ]"
