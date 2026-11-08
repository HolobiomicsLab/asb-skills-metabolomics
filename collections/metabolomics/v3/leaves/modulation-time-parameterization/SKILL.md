---
name: modulation-time-parameterization
description: Use when importing a raw GCxGC-MS NetCDF chromatogram and the modulation
  period of the instrument (e.g. 5 s for the MTBLS08 08GB.cdf file) must be supplied
  to fold the signal into the 2D-TIC.
when_to_use_negative:
- The input is already a preprocessed or folded 2D chromatogram object rather than
  a raw NetCDF file
- The data are 1D GC-MS runs with no second-dimension modulation, where folding into
  a 2D-TIC is meaningless
- The downstream analysis needs a feature/peak table rather than the raw 2D-TIC signal
  matrix
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RGCxGC
  role: Provides read_chrom, which imports the raw NetCDF chromatogram and folds it
    into the 2D-TIC using the supplied modulation time
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Backend used to read the raw chromatogram from the NetCDF file during import
- name: R
  role: Execution environment in which the RGCxGC import workflow is run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:c1c47e9b1381a0c7f07a6b1e695640ee1704b60178d198ad209eb95ac935aaba
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# modulation-time-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correctly parameterize the modulation time when importing a raw GCxGC-MS chromatogram from a NetCDF file with RGCxGC's read_chrom, so the 1D signal is folded into the correct 2D-TIC matrix whose rows correspond to modulation periods.

## When to use

Use when importing a raw GCxGC-MS NetCDF chromatogram and the modulation period of the instrument (e.g. 5 s for the MTBLS08 08GB.cdf file) must be supplied to fold the signal into the 2D-TIC.

## When NOT to use

- The input is already a preprocessed or folded 2D chromatogram object rather than a raw NetCDF file
- The data are 1D GC-MS runs with no second-dimension modulation, where folding into a 2D-TIC is meaningless
- The downstream analysis needs a feature/peak table rather than the raw 2D-TIC signal matrix

## Inputs

- Raw GCxGC-MS chromatogram in NetCDF format (e.g. 08GB.cdf from the MTBLS08/MTBLS579 dataset)
- Instrument modulation time in seconds (e.g. mod_time = 5)

## Outputs

- RGCxGC S4 chromatogram object with four slots: folded 2D-TIC matrix, first-dimension retention times, NetCDF file name, and modulation time (e.g. saved as 08GB_raw_chrom.rda)

## How to apply

Call RGCxGC's read_chrom on the raw NetCDF file (e.g. the packaged 08GB.cdf from MTBLS08), passing mod_time set to the experiment's modulation period — 5 seconds in the documented example. read_chrom accepts further parameters (name, mod_time, sam_rate, per_eval, x_cut, y_cut, verbose) that control sampling-rate handling, decimation, and retention-window cropping. The function folds the 1D total ion current into a two-dimensional Total Intensity Chromatogram (2D-TIC) whose second dimension is defined by the modulation time, and returns an S4 object with four slots: the folded 2D-TIC, the first-dimension retention times, the NetCDF file name, and the modulation time. An incorrect mod_time misassigns the second-dimension time axis and invalidates all downstream preprocessing (baseline_corr, batch_2DCOW) and MPCA (m_prcomp) steps, so verify it against the instrument method before import.

## Related tools

- **RGCxGC** (Provides read_chrom, which imports the raw NetCDF chromatogram and folds it into the 2D-TIC using the supplied modulation time) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Backend used to read the raw chromatogram from the NetCDF file during import)
- **R** (Execution environment in which the RGCxGC import workflow is run)

## Examples

```
library(RGCxGC); chrom_08 <- system.file('extdata', '08GB.cdf', package = 'RGCxGC'); raw_chrom <- read_chrom(chrom_08, mod_time = 5)
```

## Evaluation signals

- The returned S4 object exposes exactly four slots: the folded 2D-TIC chromatogram, first-dimension retention times, the NetCDF file name, and the modulation time
- The recorded modulation time slot equals the value passed to mod_time (5 for the documented 08GB.cdf example)
- The number of rows in the folded 2D-TIC is consistent with the modulation time (first-dimension run time / mod_time periods)
- The object persists correctly via save() as an .rda file (e.g. 08GB_raw_chrom.rda) and reloads without loss
- Downstream functions (baseline_corr, m_prcomp) accept the imported object without dimension errors

## Limitations

- mod_time must be known from the instrument method; the article does not describe automatic detection of the modulation period from the NetCDF metadata
- Additional read_chrom parameters (sam_rate, per_eval, x_cut, y_cut) may need tuning per dataset; defaults are not discussed in detail in the surfaced evidence
- No changelog is available for the package ('No changelog found'), so version-specific behavior changes in import cannot be traced

## Evidence

- [methods] Reading the packaged 08GB.cdf file with read_chrom(chrom_08, mod_time = 5) produces the raw MTBLS08 GCxGC S4 object: [evidence span withheld — anchor status: not_found]
- [methods] documented as having four slots: the folded 2D-TIC chromatogram, first-dimension retention times, the NetCDF file name, and the modulation time: [evidence span withheld — anchor status: not_found]
    ]"
