---
name: gcxgc-raw-data-import-cdf
description: Use when raw GC×GC–TOFMS data are stored as CDF/netCDF files and must
  be folded into a 4D samples × m/z × RT1 × RT2 array before peak picking; requires
  a known modulation time (e.g., 2.5 s) and acquired m/z range (e.g., 35–440).
when_to_use_negative:
- Input is already a peak table or feature-area matrix (e.g., ChromaTOF-exported peak
  areas) — Module I is only for raw CDF files
- 'Data are 1D GC–MS without modulation: there is no modulation time to cut the RT1
  axis at, so the 4D fold is undefined'
- Data are in mzML/mzXML or other non-CDF vendor formats not handled by readFolderCDF
edam_operation: http://edamontology.org/operation_0279
edam_topics: []
tools:
- name: GcDUO
  role: readFolderCDF reads CDF folders, folds scans at the modulation period, and
    returns the 4D GcDuo object; visualizeChrom3D/visualizeChrom2D validate the import
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Runtime environment in which GcDUO is implemented and the import is executed
  repo: https://github.com/mariallr/GcDuo
- name: EBImage
  role: Required Bioconductor dependency installed before GcDUO (used by visualization
    routines)
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:296fb70f311b5f1460ede641176182b5fa741c3037370899ec76798994c86e65
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
  - dplyr
  - ncdf4
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gcxgc-raw-data-import-cdf

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Imports raw GC×GC–MS CDF (netCDF) files with GcDUO's readFolderCDF and folds the 2D scan data into the 4D sample × m/z × RT1 × RT2 tensor (Module I), producing the GcDuo S3 object required by all downstream peak-picking and deconvolution steps.

## When to use

Use when raw GC×GC–TOFMS data are stored as CDF/netCDF files and must be folded into a 4D samples × m/z × RT1 × RT2 array before peak picking; requires a known modulation time (e.g., 2.5 s) and acquired m/z range (e.g., 35–440).

## When NOT to use

- Input is already a peak table or feature-area matrix (e.g., ChromaTOF-exported peak areas) — Module I is only for raw CDF files
- Data are 1D GC–MS without modulation: there is no modulation time to cut the RT1 axis at, so the 4D fold is undefined
- Data are in mzML/mzXML or other non-CDF vendor formats not handled by readFolderCDF

## Inputs

- Folder of raw CDF (netCDF) files from comprehensive 2D GC–MS acquisition
- Modulation time in seconds (second-dimension period, e.g., 2.5 s)
- Acquired m/z fragment range (e.g., c(35, 440))

## Outputs

- GcDuo S3 object with $data: 4D array (samples × m/z fragments × RT1 × RT2)
- $time1d: first-dimension retention time values
- $time2d: second-dimension retention time values
- $mz: m/z fragments present in the array
- $files: the CDF files contained in the object

## How to apply

Point GcDuo::readFolderCDF at the folder of CDF files, supplying the modulation time for the second chromatographic dimension (e.g., 2.5 s) and the acquired m/z fragment range (e.g., 35–440). For each file, the module extracts the scan_acquisition_time, intensity_values, mass_values and point_count vectors; mass_values are rounded to integers when data are non-nominal-resolution and aligned to the requested m/z range. The 2D scan data are cut along the first-dimension time axis at the user-supplied modulation period, producing the sample × m/z × RT1 × RT2 tensor stored in $data. Spot-check the fold visually with visualizeChrom3D (contour of one sample, e.g., sampleNum = 5) and visualizeChrom2D (TIC comparison across samples) to confirm the characteristic modulation pattern, then save the GcDuo object (e.g., gcduo_object.rds) for downstream procesData peak picking.

## Related tools

- **GcDUO** (readFolderCDF reads CDF folders, folds scans at the modulation period, and returns the 4D GcDuo object; visualizeChrom3D/visualizeChrom2D validate the import) — https://github.com/mariallr/GcDuo
- **R** (Runtime environment in which GcDUO is implemented and the import is executed) — https://github.com/mariallr/GcDuo
- **EBImage** (Required Bioconductor dependency installed before GcDUO (used by visualization routines))

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440))
```

## Evaluation signals

- The returned GcDuo object's $files slot contains every CDF file in the input folder
- $data is a 4D array with dimensions ordered samples × m/z × RT1 × RT2, and $mz matches the requested m/z range (e.g., 35–440)
- visualizeChrom3D contour of a single sample shows the characteristic GC×GC modulation pattern for the given modulation time (e.g., 2.5 s)
- visualizeChrom2D TIC overlays across samples show consistent first-dimension retention-time coverage and correct alignment of modulation cuts

## Limitations

- A wrong modulationTime misfolds the RT1 axis — the modulation period must match the acquisition method exactly
- mass_values are rounded to integers for non-nominal-resolution data, so high-resolution m/z information is not preserved at import
- R is not optimized for large memory loads, so very large CDF batches may require high-performance computing environments

## Evidence
