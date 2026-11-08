---
name: chromatogram-cdf-file-import
description: Use when raw GCxGC-MS acquisition data are stored as NetCDF/CDF files
  (e.g. 08GB.cdf, 09GB.cdf from MetaboLights MTBLS579) and must be converted into
  RGCxGC 2D-TIC chromatogram objects before preprocessing or multivariate analysis.
when_to_use_negative:
- Data are already preprocessed chromatograms or extracted feature/peak tables rather
  than raw NetCDF files
- Data are 1D GC-MS or LC-MS runs not acquired in comprehensive two-dimensional (GCxGC)
  mode, since the folding step assumes modulated 2D data
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: RGCxGC
  role: Provides read_chrom, which imports the raw chromatogram from a NetCDF file
    and folds it into a 2D-TIC object
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Underlying R library used to read the NetCDF-formatted chromatogram files
- name: R
  role: Runtime environment in which RGCxGC and read_chrom are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:e90bc636131566c0c3c8cbef2e3bac7ab13d399390c01826eedc9bb677f58745
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
  - ptw
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatogram-cdf-file-import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import a raw GCxGC chromatogram from a NetCDF/CDF file with the RGCxGC package's read_chrom, folding the one-dimensional raw signal into a two-dimensional Total Intensity Chromatogram (2D-TIC) object that downstream preprocessing (smoothing, baseline correction, 2D-COW alignment) and MPCA can consume.

## When to use

Use when raw GCxGC-MS acquisition data are stored as NetCDF/CDF files (e.g. 08GB.cdf, 09GB.cdf from MetaboLights MTBLS579) and must be converted into RGCxGC 2D-TIC chromatogram objects before preprocessing or multivariate analysis.

## When NOT to use

- Data are already preprocessed chromatograms or extracted feature/peak tables rather than raw NetCDF files
- Data are 1D GC-MS or LC-MS runs not acquired in comprehensive two-dimensional (GCxGC) mode, since the folding step assumes modulated 2D data

## Inputs

- NetCDF/CDF raw chromatogram files (e.g. 08GB.cdf, 09GB.cdf from package example data)
- Instrument parameters: modulation time, sampling rate (read_chrom arguments mod_time, sam_rate)
- Optional retention-time cut ranges (x_cut, y_cut)

## Outputs

- Two-dimensional Total Intensity Chromatogram (2D-TIC) objects in R (e.g. MTBLS08, MTBLS09)
- Named lists of chromatogram objects for batch alignment workflows

## How to apply

Load the RGCxGC package in R and call read_chrom on each NetCDF file; the function folds the raw 1D signal into a two-dimensional Total Intensity Chromatogram (2D-TIC) using the modulation time and sampling rate. read_chrom exposes tuning parameters (name, mod_time, sam_rate, per_eval, x_cut, y_cut, verbose) so you can set the modulation period, sampling rate, and optionally trim retention-time ranges during import. Imported chromatograms are returned as named R objects (e.g. MTBLS08, MTBLS09) that feed directly into baseline_corr, batch_2DCOW/twod_cow alignment, and m_prcomp MPCA. The resulting objects can also be grouped into named lists (e.g. batch_samples with Chrom1/Chrom2/Chrom3) for batch operations. RNetCDF handles the underlying NetCDF file reading.

## Related tools

- **RGCxGC** (Provides read_chrom, which imports the raw chromatogram from a NetCDF file and folds it into a 2D-TIC object) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Underlying R library used to read the NetCDF-formatted chromatogram files)
- **R** (Runtime environment in which RGCxGC and read_chrom are executed)

## Examples

```
MTBLS08 <- read_chrom("08GB.cdf"); MTBLS09 <- read_chrom("09GB.cdf")
```

## Evaluation signals

- Each CDF file yields a single 2D-TIC chromatogram object with expected 2D dimensions set by the modulation time and sampling rate parameters
- Subsequent plot of the imported chromatogram (e.g. via the filled.contour-based plotting in RGCxGC) renders a plausible 2D contour chromatogram rather than a flat or misaligned signal
- Downstream steps (baseline_corr, batch_2DCOW, m_prcomp) accept the object without dimension errors, confirming the fold was correct
- Named chromatograms (e.g. MTBLS08, MTBLS09) can be assembled into a named list and aligned against a reference, matching the vignette's batch example structure

## Limitations

- Import assumes NetCDF-formatted raw GCxGC files; other vendor formats are not supported by this step
- Correct folding depends on supplying the right modulation time and sampling rate; wrong values will distort the 2D-TIC dimensions
- Optional x_cut/y_cut trimming applied at import removes data permanently, which affects all downstream preprocessing and alignment

## Evidence

- [intro] Import raw chromatogram from a NetCDF file and fold into the two-dimensional Total Intensity Chromatogram (2D-TIC): "the raw chromatogram is importing from a **NetCDF** file and is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC)"
- [abstract] read_chrom function signature: ""read_chrom": [
      "name",
      "mod_time",
      "sam_rate",
      "per_eval",
      "x_cut",
      "y_cut",
      "verbose"
    ]"
- [readme] RGCxGC purpose for two-dimensional GC data: "The goal of RGCxGC is to provide an easy-to-use platform to analyze
two-dimensional gas chromatography data."
