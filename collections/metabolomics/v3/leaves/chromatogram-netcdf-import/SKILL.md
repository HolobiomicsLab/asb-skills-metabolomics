---
name: chromatogram-netcdf-import
description: Use when the starting point of a GCxGC-MS analysis is a raw instrument
  chromatogram stored in NetCDF (e.g. a .cdf file) and it must be converted into a
  folded 2D-TIC object before preprocessing or multivariate analysis.
when_to_use_negative:
- The chromatogram is already imported and folded (an RGCxGC chromatogram object or
  an .rda/.rds file of one) — re-importing would discard prior preprocessing such
  as baseline correction or smoothing.
- Data are two-dimensional chromatograms already stored in RGCxGC's unfolded matrix
  form for supervised mixOmics modeling via unfold_chrom, not raw NetCDF instrument
  files.
- The file is not a NetCDF chromatogram (e.g. mzML or plain spectra tables) — read_chrom's
  RNetCDF-based reader is specific to NetCDF GCxGC files.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: RGCxGC
  role: Provides the read_chrom function that imports the NetCDF chromatogram and
    folds it into the 2D-TIC S4 object
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Underlying R package used by read_chrom to access and read the raw NetCDF
    file
- name: R
  role: Runtime environment in which the RGCxGC import workflow is executed and the
    resulting chromatogram object is saved
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:a1456b2b23080cbe7ccef3fde1d8418e9c7374c77ed2cf27f67cb12861910a15
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# chromatogram-netcdf-import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import a raw comprehensive two-dimensional gas chromatography–mass spectrometry (GCxGC-MS) chromatogram from a NetCDF file with the RGCxGC read_chrom function and fold it into the two-dimensional Total Intensity Chromatogram (2D-TIC) representation, producing an S4 chromatogram object that carries first-dimension retention times, file name, and modulation time.

## When to use

Use when the starting point of a GCxGC-MS analysis is a raw instrument chromatogram stored in NetCDF (e.g. a .cdf file) and it must be converted into a folded 2D-TIC object before preprocessing or multivariate analysis.

## When NOT to use

- The chromatogram is already imported and folded (an RGCxGC chromatogram object or an .rda/.rds file of one) — re-importing would discard prior preprocessing such as baseline correction or smoothing.
- Data are two-dimensional chromatograms already stored in RGCxGC's unfolded matrix form for supervised mixOmics modeling via unfold_chrom, not raw NetCDF instrument files.
- The file is not a NetCDF chromatogram (e.g. mzML or plain spectra tables) — read_chrom's RNetCDF-based reader is specific to NetCDF GCxGC files.

## Inputs

- Raw GCxGC-MS chromatogram in NetCDF format (e.g. .cdf file such as 08GB.cdf from MetaboLights study MTBLS579)
- Modulation time for the first dimension (seconds), passed as mod_time

## Outputs

- RGCxGC S4 chromatogram object with slots: folded 2D-TIC chromatogram, first-dimension retention times, NetCDF file name, and modulation time
- Saved .rda (e.g. 08GB_raw_chrom.rda) for downstream preprocessing and MPCA steps

## How to apply

Load the RGCxGC package in R and call read_chrom on the NetCDF file (e.g. the packaged 08GB.cdf from MetaboLights study MTBLS579), supplying the modulation time via mod_time (5 for the MTBLS08 example). read_chrom imports the raw chromatogram from the NetCDF file — relying on RNetCDF for file access — and folds the one-dimensional MS signal into the two-dimensional Total Intensity Chromatogram (2D-TIC) by segmenting on the modulation time. Optional arguments name, sam_rate, per_eval, x_cut, and y_cut let you control sample naming, sampling rate, percent evaluated, and trimming of first/second-dimension retention ranges. Verify the returned S4 object exposes the expected four slots: the folded 2D-TIC chromatogram, first-dimension retention times, the NetCDF file name, and the modulation time. Only after this import step should the downstream RGCxGC workflow (baseline correction, smoothing, 2D-COW peak alignment, m_prcomp MPCA) be applied.

## Related tools

- **RGCxGC** (Provides the read_chrom function that imports the NetCDF chromatogram and folds it into the 2D-TIC S4 object) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Underlying R package used by read_chrom to access and read the raw NetCDF file)
- **R** (Runtime environment in which the RGCxGC import workflow is executed and the resulting chromatogram object is saved)

## Examples

```
R: chrom_08 <- RGCxGC::read_chrom("08GB.cdf", mod_time = 5); save(chrom_08, file = "08GB_raw_chrom.rda")
```

## Evaluation signals

- The returned object is an RGCxGC S4 chromatogram whose slots include the folded 2D-TIC, first-dimension retention times, the NetCDF file name, and the modulation time.
- The modulation time supplied to read_chrom matches the instrument's true modulation period (5 s for the MTBLS08 example); a mismatch would mis-fold the 2D-TIC dimensions.
- The folded 2D-TIC matrix has dimensions consistent with first-dimension retention range and the second-dimension modulation window after any x_cut/y_cut trimming.
- Saving the result (e.g. 08GB_raw_chrom.rda) and reloading it yields an object identical to the in-session import, confirming a complete, uncorrupted import.
- Downstream steps (baseline_corr, smoothing, 2D-COW, m_prcomp) run without dimension errors on the imported object, indicating the fold was performed correctly.

## Limitations

- read_chrom assumes the NetCDF file encodes a single raw GCxGC-MS chromatogram with a known constant modulation time; incorrect mod_time silently produces a mis-folded 2D-TIC.
- x_cut and y_cut trimming must be supplied by the user; no automatic retention-range detection is described in the article or README.
- The import is specific to NetCDF-formatted GCxGC data; other vendor formats are not supported by read_chrom.

## Evidence

- [intro] NetCDF import and 2D-TIC folding: "the raw chromatogram is importing from a **NetCDF** file and is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC)"
- [abstract] read_chrom function signature: ""read_chrom": [
      "name",
      "mod_time",
      "sam_rate",
      "per_eval",
      "x_cut",
      "y_cut",
      "verbose"
    ]"
- [methods] Task card workflow for importing 08GB.cdf: [evidence span withheld — anchor status: not_found]
- [methods] S4 object slot documentation from task card: [evidence span withheld — anchor status: not_found]
- [readme] RNetCDF dependency from README: "The goal of RGCxGC is to provide an easy-to-use platform to analyze
two-dimensional gas chromatography data."
