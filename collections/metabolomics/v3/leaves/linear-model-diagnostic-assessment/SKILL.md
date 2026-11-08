---
name: linear-model-diagnostic-assessment
description: Use when a series of Thermo .raw system-suitability or QC files (e.g.
  the ~12 h longitudinal FGCZ Orbitrap series) contains an iRT peptide mix (Biognosys)
  and you need to verify that retention times remain highly linear and comparable
  across instruments and time.
when_to_use_negative:
- The input data are already aggregated/feature tables (e.g. MaxQuant or Skyline outputs)
  rather than raw .raw files with iRT spectra — the workflow here reads raw vendor
  files via rawrr
- The raw files do not contain the iRT peptide mix or MS2 spectra of the iRT peptides,
  so no observed-vs-indexed RT pairs can be regressed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Reads spectral/retention-time data from proprietary Thermo Fisher Scientific
    Orbitrap .raw files (wraps RawFileReader) to obtain iRT peptide retention times
    per system suitability run
  repo: https://github.com/fgcz/rawrr
- name: R
  role: Statistical environment in which the per-file lm() regression of observed
    RT vs iRT is fit and slope/intercept/R squared are extracted
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr to access spectral data logged in
    Thermo Fisher Scientific raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:624c37c29e6d658b066005cedf04e6172642a5c108e9f011365521251fda95bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# linear-model-diagnostic-assessment

## Summary

Fits a per-file linear regression of observed iRT peptide retention times against indexed iRT values and inspects slope, intercept, and R squared to judge LC-RT linearity and instrument stability across Thermo Orbitrap system suitability runs.

## When to use

Use when a series of Thermo .raw system-suitability or QC files (e.g. the ~12 h longitudinal FGCZ Orbitrap series) contains an iRT peptide mix (Biognosys) and you need to verify that retention times remain highly linear and comparable across instruments and time.

## When NOT to use

- The input data are already aggregated/feature tables (e.g. MaxQuant or Skyline outputs) rather than raw .raw files with iRT spectra — the workflow here reads raw vendor files via rawrr
- The raw files do not contain the iRT peptide mix or MS2 spectra of the iRT peptides, so no observed-vs-indexed RT pairs can be regressed

## Inputs

- Thermo Fisher Scientific .raw system suitability files containing the Biognosys iRT peptide mix (e.g. longitudinal FGCZ series recorded approximately every 12 h on different Orbitrap instruments; example file EH4547 from the Bioconductor tartare package / MassIVE MSV000086542)
- iRT indexed retention time reference values for the Biognosys iRT peptide mix

## Outputs

- Per-file linear regression model objects (slope, intercept, R squared)
- irt_regression_summary.csv — table of per-file slope, intercept, and R squared ordered by acquisition time

## How to apply

Extract the iRT peptide mix (Biognosys) retention times from each system suitability rawfile using the rawrr reader family functions, e.g. after fetching the tartare/ExperimentHub file EH4547 and copying it to a local .raw file. For each rawfile, fit a linear regression of observed retention time against iRT indexed retention times with R's lm(). Extract the slope, intercept, and R squared from each per-file model, assemble the per-file statistics into a single table ordered by acquisition time, and write it as irt_regression_summary.csv. Diagnose model quality by inspecting the R squared values: high R squared on every instrument confirms that RTs behave highly linearly across the ~12 h sampling interval and that the regression models are stable across different Orbitrap instruments.

## Related tools

- **rawrr** (Reads spectral/retention-time data from proprietary Thermo Fisher Scientific Orbitrap .raw files (wraps RawFileReader) to obtain iRT peptide retention times per system suitability run) — https://github.com/fgcz/rawrr
- **R** (Statistical environment in which the per-file lm() regression of observed RT vs iRT is fit and slope/intercept/R squared are extracted)
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr to access spectral data logged in Thermo Fisher Scientific raw files) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
Rscript -e 'eh <- ExperimentHub::ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(rawfile); m <- lm(observed_rt ~ irt, data = irt_table); summary(m)$r.squared'
```

## Evaluation signals

- Each per-file model yields a finite slope and intercept and an R squared close to 1, confirming observed RTs behave highly linearly against indexed iRT values
- irt_regression_summary.csv contains one row per rawfile in the series, ordered by acquisition time, with slope, intercept, and R squared columns
- R squared values are consistent across the different Orbitrap instruments over the ~12 h longitudinal sampling interval, indicating iRT regression stability
- The same regression recomputed with rawrr reproduces the iRT regression models originally demonstrated for the FGCZ system suitability rawfiles

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version, which can affect extracted values
- On Windows, the decimal symbol must be configured as '.' for mass and intensity vectors to have correct lengths
- rawrr requires the rawrr.exe runtime (installed via rawrr::installRawrrExe()) and exchanges data with the C# layer via file I/O
- Parallel computation support is a planned next step, not present, so large longitudinal series are processed sequentially

## Evidence

- [other] Use rawrr reader family functions to extract iRT peptide retention times and fit per-file lm() regressions: [evidence span withheld — anchor status: not_found]
- [other] High R squared confirms RT linearity across instruments and time: [evidence span withheld — anchor status: not_found]
