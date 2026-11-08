---
name: regression-coefficient-verification
description: Use when reproducing a published linear calibration/regression whose
  reported coefficients, slope/intercept, and R-squared must be re-derived from raw-derived
  tabular data and compared against stated values.
when_to_use_negative:
- The relationship to verify is non-linear or the article reports a model other than
  a simple rt ~ iRTscore OLS line
- The retention-time table is not yet available (XICs for iRT precursors with 10 ppm
  MS1 tolerance have not been computed first)
- The goal is raw data access itself (e.g. reading spectra or chromatograms from Thermo
  .raw files) rather than verifying the downstream regression coefficients
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: R
  role: stats::lm and summary.lm are used to fit rt ~ iRTscore and extract intercept,
    slope, and R-squared
- name: rawrr
  role: 'Produces the upstream input: extracted ion chromatograms for iRT peptide
    precursors with readChromatogram(type = ''xic'', tol = 10, filter = ''ms'') from
    Thermo .raw files'
  repo: https://github.com/fgcz/rawrr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:4ed30c59db2253295cce9010ed05ea2af50e585e7143df29faa39c9dc8f5c491
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - rawrr
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# regression-coefficient-verification

## Summary

Fit an ordinary least squares calibration line (here rt ~ iRTscore for 11 Biognosys iRT peptides) in R and verify the reported coefficients and goodness-of-fit against the article's published model, rt = 17.85 + 0.11 * score with R-squared 0.99.

## When to use

Use when reproducing a published linear calibration/regression whose reported coefficients, slope/intercept, and R-squared must be re-derived from raw-derived tabular data and compared against stated values.

## When NOT to use

- The relationship to verify is non-linear or the article reports a model other than a simple rt ~ iRTscore OLS line
- The retention-time table is not yet available (XICs for iRT precursors with 10 ppm MS1 tolerance have not been computed first)
- The goal is raw data access itself (e.g. reading spectra or chromatograms from Thermo .raw files) rather than verifying the downstream regression coefficients

## Inputs

- Table of observed retention times for the 11 iRT peptides from XICs computed with 10 ppm mass tolerance on MS1-level scans (output of task_001)
- Biognosys iRT scores for the 11 iRT peptides (e.g. LGGNEQVTR)

## Outputs

- irt_calibration_summary.csv containing fitted intercept, slope, R-squared, and verification verdict versus rt = 17.85 + 0.11 * score (R2 = 0.99)

## How to apply

Assemble the predictor/response table — for Use Case II of the rawrr paper this is retention times for the 11 iRT peptides obtained from XICs computed by rawrr::readChromatogram with tol = 10 ppm on MS1-level scans only (filter = 'ms'), paired with their Biognosys iRT scores. Fit stats::lm(rt ~ iRTscore) via ordinary least squares and extract intercept, slope, and R-squared from summary.lm. Compare fitted values against the reported calibration line rt = 17.85 + 0.11 * score (R-squared 0.99), treating near-coincidence of slope/intercept within reasonable rounding and R-squared ≥ 0.99 as confirmation of high linearity between iRT scores and observed retention times. Write the fitted coefficients, R-squared, and a verification verdict to an output CSV (irt_calibration_summary.csv).

## Related tools

- **R** (stats::lm and summary.lm are used to fit rt ~ iRTscore and extract intercept, slope, and R-squared)
- **rawrr** (Produces the upstream input: extracted ion chromatograms for iRT peptide precursors with readChromatogram(type = 'xic', tol = 10, filter = 'ms') from Thermo .raw files) — https://github.com/fgcz/rawrr

## Examples

```
fit <- stats::lm(rt ~ iRTscore, data = irt_table); s <- summary(fit); coef(s); s$r.squared
```

## Evaluation signals

- Fitted intercept is approximately 17.85 and slope approximately 0.11 (within rounding of the reported calibration line)
- R-squared from summary.lm is approximately 0.99, indicating highly linear iRT score vs. retention-time behavior
- The regression uses exactly 11 iRT peptide observations, not more or fewer
- Retention-time inputs trace back to XICs computed with 10 ppm tolerance restricted to MS1-level scans (filter = 'ms')
- irt_calibration_summary.csv contains all three quantities (coefficients, R-squared, verdict) before the verification is declared passed

## Limitations

- rawR was not yet submitted to Bioconductor at article time, and rawrr lacked exchangeable backends and parallel computation support, so upstream XIC extraction may be slow for large files
- rawrr requires the vendor-provided .NET RawFileReader assemblies (rawrr.exe compiled via dotnet publish for osx/win/linux), so the pipeline is platform-dependent for the raw-file reading step
- Verification only checks the reported rounded coefficients; small differences from rounding in the article (e.g. slope 0.11) are expected

## Evidence

- [results] C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms"): "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [results] this is achieved by computing an extracted ion chromatogram (XIC) for iRT peptide precursors: "this is achieved by computing an extracted ion chromatogram (XIC) for iRT peptide precursors, provided they were spiked into the sample matrix"
- [methods] The analyzed sample consists of the iRT peptide mix (Biognosys) in a tryptic BSA digest (NEB): "The analyzed sample consists of the iRT peptide mix (Biognosys) in a tryptic BSA digest (NEB)"
- [readme] The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package or serves as [MsRawFileReaderBackend]"
