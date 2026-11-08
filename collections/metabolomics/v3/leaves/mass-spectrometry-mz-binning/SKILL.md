---
name: mass-spectrometry-mz-binning
description: Use when raw NetCDF/CDF files from comprehensive 2D GC–MS runs must be
  folded into a sample-by-m/z-by-RT2-by-RT1 array and the m/z axis must be rounded
  and aligned to the acquired fragment range before peak picking or deconvolution.
when_to_use_negative:
- Input is already a peak table or feature table (e.g. exported from ChromaTOF) rather
  than raw CDF scan vectors — Module I reconstruction is unnecessary and would not
  apply.
- Data are LC–MS or 1D GC–MS without a modulation period; the RT1/RT2 folding depends
  on the second-column modulation time supplied by the user.
- Only a single-chromatogram visualization is needed with no batch processing — the
  tensor construction exists to support batch peak picking, alignment and annotation.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: readFolderCDF reads CDF vectors, bins/rounds mass_values to the requested
    m/z range, and folds scan data into the 4D sample × m/z × RT1 × RT2 array
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: implementation platform in which the CDF import, m/z integer rounding and
    tensor folding are executed
  repo: https://github.com/mariallr/GcDuo
- name: EBImage
  role: required dependency installed via BiocManager, supporting the visual spot-check
    of the imported object (visualizeChrom3D/visualizeChrom2D)
- name: ncdf4
  role: R package used to read the raw CDF file vectors (scan_acquisition_time, intensity_values,
    mass_values, point_count)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:c26f441dd8001de08a2a537cf613c07a7f58bad79a99c6bcbbe3ad731a8b3360
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - EBImage
  - GcDUO
  - R
  - ncdf4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# mass-spectrometry-mz-binning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Binning of GC×GC–MS raw CDF data onto a fixed m/z axis (e.g. 35–440) so intensities form a structured samples × m/z × RT1 × RT2 4D tensor, the required input object for GcDUO peak picking and PARAFAC/PARAFAC2 deconvolution.

## When to use

Use when raw NetCDF/CDF files from comprehensive 2D GC–MS runs must be folded into a sample-by-m/z-by-RT2-by-RT1 array and the m/z axis must be rounded and aligned to the acquired fragment range before peak picking or deconvolution.

## When NOT to use

- Input is already a peak table or feature table (e.g. exported from ChromaTOF) rather than raw CDF scan vectors — Module I reconstruction is unnecessary and would not apply.
- Data are LC–MS or 1D GC–MS without a modulation period; the RT1/RT2 folding depends on the second-column modulation time supplied by the user.
- Only a single-chromatogram visualization is needed with no batch processing — the tensor construction exists to support batch peak picking, alignment and annotation.

## Inputs

- folder of raw NetCDF (CDF) files from GC × GC–MS acquisition
- modulation time (second-dimension period, e.g. 2.5 s)
- acquired m/z fragment range (e.g. c(35, 440))

## Outputs

- GcDuo S3 object with $data: 4D array (samples × m/z fragments × RT1 × RT2)
- $time1d and $time2d: retention-time axes for the two chromatographic dimensions
- $mz: the m/z fragments in the array
- $files: the CDF files contained in the object
- saved R object (e.g. gcduo_object.rds) for downstream peak picking

## How to apply

Point GcDuo::readFolderCDF at a folder of CDF files and supply the modulation time of the second chromatographic dimension (2.5 s in the paper's example) plus the acquired m/z fragment range (35–440 in the example). For each file, the scan_acquisition_time, intensity_values, mass_values and point_count vectors are extracted from the CDF vectors, and mass_values are rounded to integers for non-nominal-resolution data and aligned to the requested m/z range. The 2D scan data are then cut along the first-dimension time axis at the user-supplied modulation period, producing the S3 object with slots $data (samples × m/z × RT1 × RT2 4D array), $time1d, $time2d, $mz and $files. Rationale: downstream peak picking (GcDuo::procesData) and PARAFAC/PARAFAC2 deconvolution require a tri-linearly foldable tensor with a consistent integer m/z axis across all samples in the batch.

## Related tools

- **GcDUO** (readFolderCDF reads CDF vectors, bins/rounds mass_values to the requested m/z range, and folds scan data into the 4D sample × m/z × RT1 × RT2 array) — https://github.com/mariallr/GcDuo
- **R** (implementation platform in which the CDF import, m/z integer rounding and tensor folding are executed) — https://github.com/mariallr/GcDuo
- **EBImage** (required dependency installed via BiocManager, supporting the visual spot-check of the imported object (visualizeChrom3D/visualizeChrom2D))
- **ncdf4** (R package used to read the raw CDF file vectors (scan_acquisition_time, intensity_values, mass_values, point_count))

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440))
```

## Evaluation signals

- The imported object contains all CDF files from the input folder ($files matches the folder listing).
- $data has the expected 4D dimensions with modes in the order samples × m/z fragments × retention 1 × retention 2, and $mz spans the requested range (e.g. 35–440).
- GcDuo::visualizeChrom3D contour plot of one sample shows the characteristic GC×GC modulation pattern, confirming the 2D-to-3D fold at the modulation period.
- GcDuo::visualizeChrom2D TIC comparison across samples (e.g. samples = 1:5) shows consistent chromatograms across the batch.
- m/z values in the object are integers aligned to the fixed axis (non-nominal-resolution masses were rounded).

## Limitations

- GcDUO is implemented in R, an environment not optimized for large memory loads or multi-core parallel processing, so building the full 4D tensor for large batches may require high-performance computing environments.
- Masses from non-nominal-resolution instruments are rounded to integers, so exact-mass information is lost at import time.
- The fold depends on a correctly supplied modulation time and m/z range; mis-specified parameters will misalign the RT1/RT2 axes or the m/z axis silently.

## Evidence
