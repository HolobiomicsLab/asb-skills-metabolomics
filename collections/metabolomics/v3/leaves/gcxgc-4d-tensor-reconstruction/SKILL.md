---
name: gcxgc-4d-tensor-reconstruction
description: Use when a folder of raw CDF/NetCDF files from comprehensive two-dimensional
  GC–MS must be folded into a structured 4D array (samples × m/z × retention time
  1 × retention time 2) as the entry point of the GcDUO workflow.
when_to_use_negative:
- Input data is already a GcDuo object or an assembled feature/peak table — Module
  I is only the raw-CDF entry point of the pipeline
- Data are one-dimensional GC–MS runs (single chromatographic dimension), for which
  no modulation time exists to cut the RT1 axis
- Raw files are in mzML or other vendor formats rather than CDF/NetCDF, since readFolderCDF
  expects the CDF scan vectors (scan_acquisition_time, intensity_values, mass_values,
  point_count)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: GcDUO
  role: Provides readFolderCDF, the Module I function that folds CDF vectors into
    the 4D tensor, plus visualizeChrom3D/visualizeChrom2D for QC plotting
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform; GcDUO is installed via devtools::install_github and
    the object is saved as .rds
- name: EBImage
  role: Required dependency for GcDUO's image-based visualization checks of the imported
    object
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:ee6527b8d0aa1dccf7c631086c59fe8194e42caadbea5e15bd1b08c3a8d8ce62
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gcxgc-4d-tensor-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a 4D sample-by-m/z-by-RT1-by-RT2 tensor from raw CDF (NetCDF) GC × GC–MS files using GcDUO's readFolderCDF, the data-import module of the GcDUO pipeline. Folding the 2D scan data at the user-supplied modulation period is the prerequisite step for all downstream GcDUO peak picking, deconvolution, and annotation.

## When to use

Use when a folder of raw CDF/NetCDF files from comprehensive two-dimensional GC–MS must be folded into a structured 4D array (samples × m/z × retention time 1 × retention time 2) as the entry point of the GcDUO workflow.

## When NOT to use

- Input data is already a GcDuo object or an assembled feature/peak table — Module I is only the raw-CDF entry point of the pipeline
- Data are one-dimensional GC–MS runs (single chromatographic dimension), for which no modulation time exists to cut the RT1 axis
- Raw files are in mzML or other vendor formats rather than CDF/NetCDF, since readFolderCDF expects the CDF scan vectors (scan_acquisition_time, intensity_values, mass_values, point_count)

## Inputs

- Folder path containing raw CDF (NetCDF) GC×GC–MS files
- Modulation time of the second chromatographic dimension in seconds (e.g. 2.5)
- Acquired m/z fragment range as a two-element vector (e.g. c(35, 440))

## Outputs

- GcDuo S3 object with $data: 4D intensity array ordered samples × m/z fragments × retention 1 × retention 2
- $time1d and $time2d: time values for the 1st and 2nd chromatographic separations
- $mz: the m/z fragments in the array
- $files: the CDF files contained in the object
- Optional contour and TIC QC plots from visualizeChrom3D / visualizeChrom2D

## How to apply

Point GcDuo::readFolderCDF at the folder of CDF files, supplying the modulation time of the second chromatographic dimension (2.5 s in the article's example) and the acquired m/z range (c(35, 440) in the example). For each file, the function reads the scan_acquisition_time, intensity_values, mass_values and point_count vectors; mass_values are rounded to integers for non-nominal-resolution data and aligned to the requested m/z range. The first-dimension time axis is then cut at the modulation period, folding the 2D scan data into the 3D GC×GC structure, which is stacked across files into the 4D tensor. Inspect the returned S3 object's slots ($data, $time1d, $time2d, $mz, $files) and run the visual spot-checks: visualizeChrom3D for a single-sample contour plot and visualizeChrom2D for a TIC comparison across samples. Save the resulting GcDuo object (e.g. gcduo_object.rds) as input to peak picking via GcDuo::procesData.

## Related tools

- **GcDUO** (Provides readFolderCDF, the Module I function that folds CDF vectors into the 4D tensor, plus visualizeChrom3D/visualizeChrom2D for QC plotting) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform; GcDUO is installed via devtools::install_github and the object is saved as .rds)
- **EBImage** (Required dependency for GcDUO's image-based visualization checks of the imported object)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440))
```

## Evaluation signals

- The returned object's $data is a 4D array with dimensions ordered samples × m/z × RT1 × RT2, matching the expected shape for all files in the folder
- $files lists every CDF file present in the input folder — no files silently dropped
- $mz spans the requested range (35–440 in the example) with integer (nominal) m/z values after rounding
- visualizeChrom3D contour plot of a single sample shows the characteristic GC×GC modulation pattern (peaks repeating every 2.5 s modulation)
- visualizeChrom2D TIC overlay across samples (e.g. samples = 1:5) shows comparable chromatograms across the batch

## Limitations

- The modulation time and m/z range must be supplied manually by the user; a wrong modulation time folds the RT1 axis incorrectly
- Non-nominal-resolution data require mass_values rounding to integers before alignment, which loses high-resolution m/z information
- R is not optimized for large memory loads or multi-core parallelism, so importing many samples may require high-performance computing environments

## Evidence
