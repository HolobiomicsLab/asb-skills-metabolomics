---
name: longitudinal-system-suitability-monitoring
description: Use when a series of Thermo Orbitrap autoQC/system-suitability .raw files
  is available across time or instruments and you must verify retention-time linearity
  (iRT calibration slope, intercept, R²) without converting to mzML.
when_to_use_negative:
- Input files are not Thermo Fisher Scientific raw files — rawrr wraps the RawFileReader
  API and only reads this vendor format
- Data has already been converted to exchange formats like mzML (e.g. via ProteoWizard/MSconvert)
  — use standard R MS backends instead
- The goal is generic LC-MS method QC plotting rather than iRT retention-time regression
  — rawDiag's LC-MS method optimization visualizations may be more appropriate
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: Reads spectral and chromatographic data (including iRT peptide retention times)
    directly from Thermo Orbitrap .raw files in R via the wrapped RawFileReader API
  repo: https://github.com/fgcz/rawrr
- name: rawR
  role: Predecessor/stand-alone package providing complete, OS-independent access
    to all spectral data logged in Thermo Fisher Scientific raw files
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET (C#) API wrapped by rawrr that performs the actual reading
    of proprietary Thermo Scientific RAW files on Windows, Linux, and MacOS
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: tartare
  role: Bioconductor data package supplying the demonstration system-suitability raw
    files (EH4547) used for the longitudinal iRT regression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:35b8cff884072fc880db8bc415d172f56ed94bb73d9aeee53d34eab58ed46193
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  - tartare
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# longitudinal-system-suitability-monitoring

## Summary

A skill for tracking LC-MS instrument stability over time by recomputing iRT (Biognosys) peptide retention-time regression models directly from Thermo Fisher Orbitrap .raw system-suitability files acquired at regular (~12 h) intervals, using the rawrr R package.

## When to use

Use when a series of Thermo Orbitrap autoQC/system-suitability .raw files is available across time or instruments and you must verify retention-time linearity (iRT calibration slope, intercept, R²) without converting to mzML.

## When NOT to use

- Input files are not Thermo Fisher Scientific raw files — rawrr wraps the RawFileReader API and only reads this vendor format
- Data has already been converted to exchange formats like mzML (e.g. via ProteoWizard/MSconvert) — use standard R MS backends instead
- The goal is generic LC-MS method QC plotting rather than iRT retention-time regression — rawDiag's LC-MS method optimization visualizations may be more appropriate

## Inputs

- Thermo Fisher Scientific Orbitrap .raw system-suitability/autoQC files recorded at ~12 h intervals on multiple instruments
- Biognosys iRT peptide mix (e.g. LGGNEQVTR, iRT indexed retention times) for calibration
- R environment with rawrr installed (rawrr.exe runtime via installRawrrExe)

## Outputs

- Per-rawfile iRT regression models fitted with lm() (slope, intercept, R squared)
- irt_regression_summary.csv — per-file statistics ordered by acquisition time

## How to apply

Install the rawrr runtime with rawrr::installRawrrExe(), which wraps the vendor RawFileReader .NET API for OS-independent access to Thermo raw files. Fetch demonstration system-suitability data via ExperimentHub (e.g. the tartare rawfile EH4547) and copy it to a local .raw file. For each rawfile in the longitudinal series, use rawrr's reader functions to extract observed retention times of the iRT peptide mix, then fit a per-file linear model lm(observed_RT ~ iRT) in R and collect slope, intercept, and R squared. Assemble per-file statistics ordered by acquisition time into irt_regression_summary.csv; high, stable R squared values across instruments confirm RTs behave linearly over the ~12 h sampling interval.

## Related tools

- **rawrr** (Reads spectral and chromatographic data (including iRT peptide retention times) directly from Thermo Orbitrap .raw files in R via the wrapped RawFileReader API) — https://github.com/fgcz/rawrr
- **rawR** (Predecessor/stand-alone package providing complete, OS-independent access to all spectral data logged in Thermo Fisher Scientific raw files) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET (C#) API wrapped by rawrr that performs the actual reading of proprietary Thermo Scientific RAW files on Windows, Linux, and MacOS) — https://github.com/thermofisherlsms/RawFileReader
- **tartare** (Bioconductor data package supplying the demonstration system-suitability raw files (EH4547) used for the longitudinal iRT regression)

## Examples

```
rawrr::installRawrrExe(); eh <- ExperimentHub::ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]])
```

## Evaluation signals

- Each per-file iRT regression yields high R squared, confirming retention times behave highly linearly on each instrument across the sampling interval
- irt_regression_summary.csv contains one row per rawfile with slope, intercept, and R² columns and is ordered by acquisition time
- The iRT peptide LGGNEQVTR can be identified and its retention time extracted (reference: MS-GF+ Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 for scan 9594, charge 2)
- rawrr.exe runtime installed successfully via rawrr::installRawrrExe() and EH4547 resolved to a local .raw path via normalizePath(eh[["EH4547"]])
- Recomputed regression parameters remain stable across different Orbitrap instruments in the longitudinal series

## Limitations

- On Windows, the decimal symbol has to be configured as a '.' or the mZ and intensity vectors returned by rawrr will have incorrect lengths
- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- Data exchange between the R and C# layers is done via file I/O; exchangeable Spectra backends (MsBackendRawFileReader) and parallel computation support were only planned at time of writing

## Evidence
