---
name: gcxgc-2dtic-folding
description: Use when raw GCxGC-MS data is provided as a NetCDF/CDF file (e.g. 08GB.cdf
  from MTBLS) and must be imported and folded into a 2D-TIC object before smoothing,
  baseline correction, or MPCA.
when_to_use_negative:
- The chromatogram has already been imported and folded — further steps belong to
  preprocessing (baseline_corr, 2D-COW alignment) or multivariate analysis, not re-import
- Data is not raw GCxGC-MS signal in NetCDF format (e.g. already-unfolded feature
  tables for supervised mixOmics models)
- A one-dimensional GC-MS chromatogram without a modulation time is provided, since
  2D folding is undefined
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RGCxGC
  role: Provides read_chrom to import the NetCDF raw chromatogram and fold it into
    the 2D-TIC S4 object
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Underlies the reading of the raw chromatogram from the NetCDF file
- name: R
  role: Runtime environment in which the RGCxGC import workflow is executed
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:07271da5a715778cfa56a552ce47c14a3d99ab2130a8423c47146ce50419edec
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gcxgc-2dtic-folding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import a raw GCxGC-MS chromatogram from a NetCDF file using RGCxGC's read_chrom and fold the 1D signal into the two-dimensional Total Intensity Chromatogram (2D-TIC) using the modulation time, the entry point of the RGCxGC preprocessing pipeline.

## When to use

Use when raw GCxGC-MS data is provided as a NetCDF/CDF file (e.g. 08GB.cdf from MTBLS) and must be imported and folded into a 2D-TIC object before smoothing, baseline correction, or MPCA.

## When NOT to use

- The chromatogram has already been imported and folded — further steps belong to preprocessing (baseline_corr, 2D-COW alignment) or multivariate analysis, not re-import
- Data is not raw GCxGC-MS signal in NetCDF format (e.g. already-unfolded feature tables for supervised mixOmics models)
- A one-dimensional GC-MS chromatogram without a modulation time is provided, since 2D folding is undefined

## Inputs

- Raw GCxGC-MS chromatogram in NetCDF/CDF format (e.g. 08GB.cdf from the MTBLS dataset)
- Modulation time in seconds (mod_time, e.g. 5)

## Outputs

- RGCxGC S4 chromatogram object holding the folded 2D-TIC matrix
- First-dimension retention times stored in the object slot
- NetCDF file name and modulation time stored in the object slots
- Saved .rda file of the raw chromatogram (e.g. 08GB_raw_chrom.rda)

## How to apply

Load the RGCxGC package in R and call read_chrom on the NetCDF file (e.g. the packaged 08GB.cdf) supplying mod_time (e.g. mod_time = 5 for MTBLS08), optionally sam_rate, per_eval, x_cut and y_cut to control sampling rate, evaluation percentage and retention-time trimming. The function reads the raw chromatogram via RNetCDF and folds it into the two-dimensional Total Intensity Chromatogram (2D-TIC) according to the modulation period. Verify the returned S4 chromatogram object contains the four documented slots: the folded 2D-TIC matrix, first-dimension retention times, the NetCDF file name, and the modulation time, then save it (e.g. as 08GB_raw_chrom.rda) for downstream preprocessing (baseline_corr, batch_2DCOW) and multivariate analysis (m_prcomp).

## Related tools

- **RGCxGC** (Provides read_chrom to import the NetCDF raw chromatogram and fold it into the 2D-TIC S4 object) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Underlies the reading of the raw chromatogram from the NetCDF file)
- **R** (Runtime environment in which the RGCxGC import workflow is executed)

## Examples

```
library(RGCxGC); chrom_08 <- read_chrom("08GB.cdf", mod_time = 5); save(chrom_08, file = "08GB_raw_chrom.rda")
```

## Evaluation signals

- The returned S4 object has the four documented slots: folded 2D-TIC chromatogram, first-dimension retention times, NetCDF file name, and modulation time
- The folded 2D-TIC dimensions are consistent with the chosen mod_time (first-dimension time axis divided into modulation periods of the given duration)
- The object's file-name slot matches the source NetCDF file (e.g. 08GB.cdf) and the modulation-time slot matches the mod_time argument (5 s for MTBLS08)
- The saved .rda file (e.g. 08GB_raw_chrom.rda) reloads and feeds downstream baseline_corr / m_prcomp without error

## Limitations

- Correct mod_time must be supplied by the user; an incorrect modulation time yields a wrongly folded 2D-TIC
- Optional parameters (sam_rate, per_eval, x_cut, y_cut) affect sampling and retention-time cuts and must match the acquisition settings of the dataset
- Import is limited to NetCDF-format raw chromatograms; other vendor formats are not covered by this workflow

## Evidence

    ]"
- [other] Task finding on the S4 object slots: [evidence span withheld — anchor status: not_found]
