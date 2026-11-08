---
name: chromatographic-modulation-folding
description: Use when raw comprehensive two-dimensional GC–MS data in CDF files must
  be folded into a 4D tensor (samples, m/z fragments, second-dimension and first-dimension
  retention time) as the entry point of an untargeted GC×GC–MS workflow.
when_to_use_negative:
- Input is already a peak table or feature matrix (e.g. exported from ChromaTOF) rather
  than raw CDF scan data — no modulation folding is needed
- Data are one-dimensional GC–MS runs without a modulation period, so the 2D-to-3D
  cut at the modulation time is not applicable
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: readFolderCDF performs the CDF import and modulation-period folding into the
    4D tensor; visualizeChrom3D/visualizeChrom2D verify the result
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and run (installed via devtools::install_github("mariallr/GcDUO"))
- name: EBImage
  role: Required dependency of the GcDUO package
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:b9c6fb67600dcb322955d342290b2ece0577eb823e36f3b015b9cf51669fb84c
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chromatographic-modulation-folding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs raw GC×GC–MS NetCDF (CDF) scan data into a 4D sample × m/z × RT2 × RT1 tensor by cutting the first-dimension time axis at the user-supplied modulation period, producing the structured GcDuo object required for downstream peak picking.

## When to use

Use when raw comprehensive two-dimensional GC–MS data in CDF files must be folded into a 4D tensor (samples, m/z fragments, second-dimension and first-dimension retention time) as the entry point of an untargeted GC×GC–MS workflow.

## When NOT to use

- Input is already a peak table or feature matrix (e.g. exported from ChromaTOF) rather than raw CDF scan data — no modulation folding is needed
- Data are one-dimensional GC–MS runs without a modulation period, so the 2D-to-3D cut at the modulation time is not applicable

## Inputs

- Folder of raw GC×GC–MS NetCDF (CDF) files
- Modulation time in seconds (e.g. 2.5 s), defining the second chromatographic dimension
- Acquired m/z fragment range (e.g. c(35, 440))

## Outputs

- GcDuo S3 object with `$data`: 4D array (samples × m/z fragments × retention 1 × retention 2)
- `$time1d` and `$time2d`: retention time axes for the 1st and 2nd chromatographic separations
- `$mz`: m/z fragments in the array; `$files`: CDF files contained in the object
- Saved GcDuo object (e.g. gcduo_object.rds) for downstream peak picking

## How to apply

Point GcDuo::readFolderCDF at the folder of CDF files, supplying the modulation time for the second chromatographic dimension (2.5 s in the example) and the acquired m/z fragment range (35–440 in the example). For each file, extract the scan_acquisition_time, intensity_values, mass_values and point_count vectors; round mass_values to integers for non-nominal-resolution data and align them to the requested m/z range. Cut the first-dimension time axis at the modulation period to fold the 2D scan data into the 3D GC×GC structure, yielding the sample × m/z × RT1 × RT2 array stored in the `$data` slot of the S3 object together with `$time1d`, `$time2d`, `$mz` and `$files`. Spot-check the folding with GcDuo::visualizeChrom3D (per-sample contour) and GcDuo::visualizeChrom2D (TIC comparison across samples) before saving the object as an .rds file for downstream peak picking.

## Related tools

- **GcDUO** (readFolderCDF performs the CDF import and modulation-period folding into the 4D tensor; visualizeChrom3D/visualizeChrom2D verify the result) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and run (installed via devtools::install_github("mariallr/GcDUO")))
- **EBImage** (Required dependency of the GcDUO package)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440))
```

## Evaluation signals

- The imported object contains all CDF files from the folder with the expected 4D dimensions (samples × m/z × RT1 × RT2)
- visualizeChrom3D contour of a single sample shows the characteristic GC×GC modulation pattern, confirming correct folding
- visualizeChrom2D TIC plots across samples (e.g. samples = 1:5) are comparable and structurally consistent
- `$time1d`/`$time2d` axes and `$mz` match the supplied modulation time (e.g. 2.5 s) and m/z range (e.g. 35–440)

## Limitations

- For non-nominal-resolution data, mass_values must be rounded to integers and aligned to the requested m/z range before folding
- R is 'not optimized for handling large memory loads or parallel processing in multiple cores', so 4D reconstruction of large batches may necessitate high-performance computing environments

## Evidence

- [results] Module I folds the extracted intensity and m/z vectors into a 4D tensor whose modes correspond, respectively, to samples, mass-to-charge ions, second-dimension time, and first-dimension time.: [evidence span withheld — anchor status: not_found]
- [methods] readFolderCDF call with modulation time and m/z range: "GcDuo::readFolderCDF(folderPath = path,
                                    modulationTime = 2.5, #the second dimension time (seconds)"
- [readme] README description of the 4D data slot ordering: "`$data`: the 4D array with  the files intensities. The order is the following: samples x m/z fragments x retention 1 x m retentio.n- 2."
