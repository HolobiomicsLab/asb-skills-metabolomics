---
name: irt-peptide-retention-time-regression
description: Use when you have Thermo Fisher .raw system-suitability or QC runs containing
  the iRT peptide mix (e.g. Biognosys standards) and need per-run RT calibration models
  or a longitudinal assessment of RT linearity across Orbitrap instruments.
when_to_use_negative:
- Input is not vendor .raw data (e.g. already-converted mzML or a peptide identification
  table) — rawrr reads proprietary Thermo raw files only
- The runs lack iRT peptides or identifiable iRT signal, so observed RTs cannot be
  matched to indexed RTs
- Goal is peak/intensity quality control or LC-MS method optimization rather than
  RT calibration — rawDiag's instrument-monitoring visualizations are the better fit
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rawrr
  role: Reads spectral and chromatographic data directly from Thermo Fisher .raw files
    via wrapped RawFileReader, extracting iRT peptide retention times from each system-suitability
    run
  repo: https://github.com/fgcz/rawrr
- name: R
  role: Environment used to fit the per-file lm() regressions of observed vs indexed
    RT and assemble the summary table
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr that provides OS-independent access
    to the raw spectral data
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: tartare
  role: Bioconductor data package supplying demonstration raw files (e.g. EH4547)
    for building and testing the regression workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:ee3a1e6765df41dc8e438b02cee6f562a542d835922e348b91f954382c6d4fff
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- irt-peptide-retention-time-calibration
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# irt-peptide-retention-time-regression

## Summary

Fit per-rawfile linear regressions of observed retention times of the Biognosys iRT peptide mix against indexed retention times, using the rawrr package to read Thermo Fisher Orbitrap .raw files directly, and assemble slope/intercept/R-squared statistics to verify RT linearity and regression stability across instruments and time.

## When to use

Use when you have Thermo Fisher .raw system-suitability or QC runs containing the iRT peptide mix (e.g. Biognosys standards) and need per-run RT calibration models or a longitudinal assessment of RT linearity across Orbitrap instruments.

## When NOT to use

- Input is not vendor .raw data (e.g. already-converted mzML or a peptide identification table) — rawrr reads proprietary Thermo raw files only
- The runs lack iRT peptides or identifiable iRT signal, so observed RTs cannot be matched to indexed RTs
- Goal is peak/intensity quality control or LC-MS method optimization rather than RT calibration — rawDiag's instrument-monitoring visualizations are the better fit

## Inputs

- Thermo Fisher Scientific .raw system-suitability files containing the Biognosys iRT peptide mix (longitudinal ~12 h series across Orbitrap instruments)
- Reference demo rawfile EH4547 from the Bioconductor tartare package (fetched via ExperimentHub)
- iRT peptide indexed retention time values (e.g. for LGGNEQVTR and other iRT peptides)

## Outputs

- Per-rawfile linear regression models (lm of observed RT vs iRT indexed RT)
- Table of per-file slope, intercept, and R squared ordered by acquisition time (e.g. irt_regression_summary.csv)

## How to apply

Fetch the reference rawfile (e.g. EH4547 from the Bioconductor tartare package via ExperimentHub) and copy it to a local .raw file, ensuring rawrr::installRawrrExe() has installed the rawrr.exe runtime built on the vendor RawFileReader API. Use the rawrr reader functions to extract observed retention times of the iRT peptide mix (e.g. LGGNEQVTR) from each system suitability rawfile in the ~12 h longitudinal series recorded on different Orbitrap instruments. For each rawfile, fit lm(observed RT ~ iRT indexed RT) and extract slope, intercept, and R squared. Assemble the per-file statistics into a single table ordered by acquisition time (e.g. irt_regression_summary.csv) and confirm R squared values show highly linear RT behaviour on each instrument.

## Related tools

- **rawrr** (Reads spectral and chromatographic data directly from Thermo Fisher .raw files via wrapped RawFileReader, extracting iRT peptide retention times from each system-suitability run) — https://github.com/fgcz/rawrr
- **R** (Environment used to fit the per-file lm() regressions of observed vs indexed RT and assemble the summary table)
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr that provides OS-independent access to the raw spectral data) — https://github.com/thermofisherlsms/RawFileReader
- **tartare** (Bioconductor data package supplying demonstration raw files (e.g. EH4547) for building and testing the regression workflow)

## Examples

```
library(ExperimentHub); eh <- ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); rt <- rawrr::readChromatogram(rawfile, type='bpc'); fit <- lm(observedRT ~ irt
```

## Evaluation signals

- Per-file R squared values are very high, confirming RTs behave highly linearly on each instrument across the ~12 h sampling interval
- Slope and intercept are recovered for every rawfile in the series and the summary table is ordered by acquisition time
- rawrr.exe runtime is correctly installed (rawrr::installRawrrExe()) and .raw files are readable on the target OS (Windows, Linux, macOS)
- Extracted iRT RTs match known iRT peptides (e.g. LGGNEQVTR identifications) rather than missing/empty scan data
- The regression recomputation is reproducible across different Orbitrap instruments in the longitudinal series

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' or mZ and intensity vectors will have incorrect lengths
- Parallel computation support is a planned next step, so reading many rawfiles serially may be slow
- rawrr requires the rawrr.exe runtime (installRawrrExe) and cross-compiled binaries via dotnet publish for osx/win/linux x64

## Evidence
