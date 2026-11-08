---
name: netcdf-chromatogram-matrix-handling
description: Use when raw GC×GC-MS chromatograms are stored in NetCDF files and must
  be loaded into R as 2D-TIC chromatogram matrices via RGCxGC's read_chrom to begin
  preprocessing and multivariate analysis.
when_to_use_negative:
- Data is already a preprocessed or folded 2D chromatogram object within RGCxGC, in
  which case the downstream preprocessing or MPCA functions should be applied directly
- The aim is supervised modeling on unfolded chromatograms; in that case unfold_chrom
  (with mixOmics) is used on the chromatogram object rather than raw NetCDF import
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RGCxGC
  role: Package whose read_chrom function reads raw chromatograms from NetCDF files
    and folds them into the 2D-TIC chromatogram matrix used by its preprocessing and
    MPCA workflow
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Backend used by RGCxGC to import the raw chromatogram from a NetCDF file
- name: R
  role: Environment in which the RGCxGC NetCDF import and 2D-TIC folding are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:2c1568cc264a66d10644d5f18cbaa1877256a59fa0c52424afe0aaa251985238
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# netcdf-chromatogram-matrix-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Importing raw comprehensive two-dimensional gas chromatography-mass spectrometry chromatograms from NetCDF files into the RGCxGC package and folding them into a two-dimensional Total Ion Current (2D-TIC) chromatogram matrix as the entry point of the RGCxGC preprocessing and multivariate analysis workflow.

## When to use

Use when raw GC×GC-MS chromatograms are stored in NetCDF files and must be loaded into R as 2D-TIC chromatogram matrices via RGCxGC's read_chrom to begin preprocessing and multivariate analysis.

## When NOT to use

- Data is already a preprocessed or folded 2D chromatogram object within RGCxGC, in which case the downstream preprocessing or MPCA functions should be applied directly
- The aim is supervised modeling on unfolded chromatograms; in that case unfold_chrom (with mixOmics) is used on the chromatogram object rather than raw NetCDF import

## Inputs

- Raw GC×GC-MS chromatogram in NetCDF file format
- Instrument acquisition parameters (modulation time, sampling rate, evaluation period)
- Optional MTBLS.rda preprocessed chromatogram files (e.g. from MetaboLights MTBLS579)

## Outputs

- 2D-TIC chromatogram object (two-dimensional Total Intensity Chromatogram matrix) in the RGCxGC package environment

## How to apply

Import the raw chromatogram from its NetCDF file with RGCxGC's read_chrom, supplying the chromatogram name and instrument acquisition parameters such as the modulation time (mod_time), sampling rate (sam_rate), and evaluation period (per_eval), and optionally trimming the chromatogram with x_cut and y_cut before folding. The importer folds the one-dimensional NetCDF signal into the two-dimensional Total Intensity Chromatogram (2D-TIC) according to the instrument's modulation parameters, producing the 2D matrix representation required by all downstream RGCxGC preprocessing steps (smoothing, baseline correction, 2D-COW alignment) and multivariate analysis (MPCA via m_prcomp). Data can also be loaded directly from deposited studies such as the MTBLS579 dataset in the MetaboLights database using the MTBLS.rda file shared in the DanielQuiroz97/MTBLS579 repository as an alternative to raw NetCDF import. Validate the import by inspecting the 2D-TIC matrix dimensions and visualization, then proceed to preprocessing within the RGCxGC package environment.

## Related tools

- **RGCxGC** (Package whose read_chrom function reads raw chromatograms from NetCDF files and folds them into the 2D-TIC chromatogram matrix used by its preprocessing and MPCA workflow) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Backend used by RGCxGC to import the raw chromatogram from a NetCDF file)
- **R** (Environment in which the RGCxGC NetCDF import and 2D-TIC folding are executed)

## Examples

```
chrom <- read_chrom(name = "08GB", mod_time = 5, sam_rate = 200, per_eval = 2)
```

## Evaluation signals

- The imported object is a 2D-TIC (two-dimensional Total Intensity Chromatogram) with the dimensions implied by the supplied mod_time, sam_rate, and per_eval acquisition parameters
- x_cut and y_cut values applied during read_chrom trim the chromatogram to the intended separation space
- Imported chromatograms are compatible with subsequent RGCxGC preprocessing functions (baseline_corr, smoothing, batch_2DCOW) without further reshaping
- Multiple chromatograms imported with consistent parameters can be combined for MPCA via m_prcomp

## Limitations

- NetCDF import depends on correctly specified acquisition parameters (mod_time, sampling rate, evaluation period); incorrect values will misfold the 2D-TIC matrix
- The vignette workflow (08GB chromatogram) assumes NetCDF inputs; preprocessed chromatograms from deposited studies are distributed as .rda files instead

## Evidence

    ]"
