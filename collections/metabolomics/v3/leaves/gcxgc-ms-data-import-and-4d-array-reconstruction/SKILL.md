---
name: gcxgc-ms-data-import-and-4d-array-reconstruction
description: Use when raw GC×GC–MS data are supplied as CDF (NetCDF) files and the
  analysis goal requires building a 4D data array in R with GcDUO as the first step
  of an untargeted peak-picking/deconvolution pipeline.
when_to_use_negative:
- Input is already a processed peak/feature table (e.g. a ChromaTOF peak-area export)
  rather than raw CDF files — import/deconvolution is unnecessary.
- Data are 1D GC–MS runs rather than comprehensive two-dimensional GC×GC with a defined
  modulation time, since folding 2D→3D requires a second-dimension time.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: R package whose readFolderCDF function reads the CDF files and reconstructs
    the 4D GcDuoObject; visualizeChrom3D/visualizeChrom2D provide visual checks of
    the reconstruction.
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and run; devtools and BiocManager (for
    the EBImage dependency) handle installation.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:6222205f89b36383e38d53f524c2765087d82dbd265aee49edc129a1c83bb8da
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
  - GcDUO
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gcxgc-ms-data-import-and-4d-array-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct raw GC×GC–MS CDF files into the GcDUO 4D intensity array (samples × m/z × 1st-dimension retention × 2nd-dimension retention) using readFolderCDF with the correct modulation time and acquired m/z range, optionally verifying the reconstruction with contour/TIC visualizations before downstream peak picking.

## When to use

Use when raw GC×GC–MS data are supplied as CDF (NetCDF) files and the analysis goal requires building a 4D data array in R with GcDUO as the first step of an untargeted peak-picking/deconvolution pipeline.

## When NOT to use

- Input is already a processed peak/feature table (e.g. a ChromaTOF peak-area export) rather than raw CDF files — import/deconvolution is unnecessary.
- Data are 1D GC–MS runs rather than comprehensive two-dimensional GC×GC with a defined modulation time, since folding 2D→3D requires a second-dimension time.

## Inputs

- Folder of raw GC×GC–MS CDF (NetCDF) files, e.g. the fruitybeer dataset
- Instrument method metadata: modulation (second-column) time in seconds
- Acquired m/z fragment range (e.g. c(35, 440))

## Outputs

- GcDuoObject S3 object with $data (4D array: samples × m/z fragments × retention-1 × retention-2), $time1d, $time2d, $mz, $files
- Visual quality-check plots: contour plot via visualizeChrom3D(GcDuoObject, sampleNum = 5) and TIC comparison via visualizeChrom2D(GcDuoObject, samples = 1:5)

## How to apply

Install GcDUO via devtools::install_github("mariallr/GcDUO") and first load the required EBImage dependency through BiocManager. Call GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440)), where modulationTime is the second-dimension time in seconds for the specific instrument method and mzRange must match the m/z fragments actually acquired (here 35–440); the folder passed should contain the batch of raw CDF files (e.g. the fruitybeer dataset). The returned GcDuoObject is an S3 object whose $data slot holds the 4D array ordered as samples × m/z fragments × retention-1 × retention-2, alongside $time1d, $time2d, $mz, and $files. Rationale: incorrect modulationTime misfolds the 2D data into the 3D chromatographic dimensions, and a wrong mzRange truncates or pads the spectral axis, so both parameters must be checked against the acquisition method before proceeding to procesData for peak picking.

## Related tools

- **GcDUO** (R package whose readFolderCDF function reads the CDF files and reconstructs the 4D GcDuoObject; visualizeChrom3D/visualizeChrom2D provide visual checks of the reconstruction.) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and run; devtools and BiocManager (for the EBImage dependency) handle installation.)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440))
```

## Evaluation signals

- GcDuoObject$data has exactly four dimensions ordered as samples × m/z fragments × retention-1 × retention-2, with length($files) equal to the number of CDF files in the folder.
- visualizeChrom3D(GcDuoObject, sampleNum = 5) renders a sensible contour plot (modulated peaks aligned along modulation periods) for the chosen sample.
- visualizeChrom2D(GcDuoObject, samples = 1:5) shows comparable total-ion-chromatogram contour plots across samples, with no shifted or folded retention axes indicating a wrong modulationTime.
- The $mz slot spans the declared mzRange (35–440) and the $time1d/$time2d vectors are consistent with the acquisition method.

## Limitations

- The modulation time must be supplied by the user and matches the second-column separation time; an incorrect value corrupts the 4D reconstruction.
- No software version number for GcDUO or R is stated in the paper; the GitHub release is labelled 'Beta/Github release'.
- GcDUO is implemented in R, 'an environment that is not optimized for handling large memory loads', so large batches may require high-performance computing environments.

## Evidence

- [results] Module I reconstructs the 4D matrix from the raw data, obtaining the first GcDUO object.: [evidence span withheld — anchor status: not_found]
- [readme] Installation instructions for GcDUO and EBImage: [evidence span withheld — anchor status: not_found]
