---
name: lc-gradient-rt-interpretaion
description: Use when a Thermo Fisher .raw file contains a run spiked with iRT peptides
  and you need to calibrate/interpret retention times by regressing extracted RTs
  against iRT scores, or to verify gradient performance (slope ~ 1.5 %B/min).
when_to_use_negative:
- The raw file does not contain the iRT peptide standard, so no chromatogram traces
  for the iRT masses can be extracted
- Retention times are already aligned/calibrated against a reference (no iRT regression
  coefficients are needed)
- The goal is peptide identification from MS2 spectra (e.g. MS-GF+ assignment of LGGNEQVTR),
  not retention-time regression
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: rawrr
  role: Wraps the RawFileReader .NET API to install the runtime (rawrr::installRawrrExe)
    and compute MS1 extracted ion chromatograms (rawrr::readChromatogram) used for
    the RT extraction feeding the iRT regression
  repo: https://github.com/fgcz/rawrr
- name: R
  role: Statistical environment in which lm() is used to fit the iRT regression and
    to report R-squared, slope, and intercept
  repo: https://www.r-project.org/
- name: rawR
  role: Provides the OS-independent direct access to the spectral/XIC data in the
    Thermo raw file demonstrated in the manuscript from which this workflow derives
  repo: https://github.com/fgcz/rawR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:2b4b1e8a0277c09e5330f6f94376caf695a169af6d20259afb90c76f19e28684
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-gradient-rt-interpretaion

## Summary

Fits a linear iRT regression (rt = a + b * iRT score) from retention times extracted from iRT-peptide chromatogram traces and interprets the coefficients: the slope (b) directly reflects the LC gradient change rate and the intercept (a) equals the predicted RT of the iRT peptide GAGSSEPVTGLDAK, defined to have iRT score 0.

## When to use

Use when a Thermo Fisher .raw file contains a run spiked with iRT peptides and you need to calibrate/interpret retention times by regressing extracted RTs against iRT scores, or to verify gradient performance (slope ~ 1.5 %B/min).

## When NOT to use

- The raw file does not contain the iRT peptide standard, so no chromatogram traces for the iRT masses can be extracted
- Retention times are already aligned/calibrated against a reference (no iRT regression coefficients are needed)
- The goal is peptide identification from MS2 spectra (e.g. MS-GF+ assignment of LGGNEQVTR), not retention-time regression

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file containing iRT peptide standard runs (e.g. 20181113_010_autoQC01.raw, MassIVE MSV000086542 / ExperimentHub EH4547)
- List of iRT peptide target masses and their iRT scores (GAGSSEPVTGLDAK has score 0)
- rawrr runtime (installed via rawrr::installRawrrExe())

## Outputs

- Per-peptide RT values taken at the maxima of the MS1 XIC intensity traces (10 ppm tolerance)
- Fitted linear model of rt = a + b * iRT score with R-squared, slope (~1.5 %B/min gradient rate), and intercept (predicted RT of GAGSSEPVTGLDAK)
- irt_regression_report.txt and a plot of RT vs iRT values with the fitted regression line

## How to apply

Fetch the example raw file 20181113_010_autoQC01.raw via ExperimentHub (EH4547, from MassIVE MSV000086542) and install the rawrr runtime with rawrr::installRawrrExe(). Compute extracted ion chromatograms for the iRT peptide masses with rawrr::readChromatogram using a 10 ppm tolerance around each target mass and only MS1-level scans (filter = 'ms'). For each iRT peptide, determine the retention time at the maximum of its intensity trace. Fit lm(rt ~ irt_score) in R; report R-squared, slope, and intercept in irt_regression_report.txt and plot RT vs iRT with the fitted line. Accept the fit only when linearity is high (high R-squared), the slope is consistent with the ~1.5 %B/min gradient change rate, and the intercept equals the predicted RT of GAGSSEPVTGLDAK (iRT score 0).

## Related tools

- **rawrr** (Wraps the RawFileReader .NET API to install the runtime (rawrr::installRawrrExe) and compute MS1 extracted ion chromatograms (rawrr::readChromatogram) used for the RT extraction feeding the iRT regression) — https://github.com/fgcz/rawrr
- **R** (Statistical environment in which lm() is used to fit the iRT regression and to report R-squared, slope, and intercept) — https://www.r-project.org/
- **rawR** (Provides the OS-independent direct access to the spectral/XIC data in the Thermo raw file demonstrated in the manuscript from which this workflow derives) — https://github.com/fgcz/rawR

## Examples

```
eh <- ExperimentHub(); EH4547 <- normalizePath(eh[["EH4547"]]); rawrr::installRawrrExe()
```

## Evaluation signals

- The linear model shows high R-squared (high linearity between iRT scores and extracted RTs)
- The fitted slope b is consistent with the ~1.5 %B/min gradient change rate of the LC method
- The fitted intercept a equals the predicted RT of the iRT peptide GAGSSEPVTGLDAK (iRT score 0)
- XICs were computed with a 10 ppm tolerance using only MS1-level scans (filter = 'ms'), matching the paper's parameters
- The report irt_regression_report.txt contains R-squared, slope, and intercept, plus an RT-vs-iRT plot with the fitted line

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version, so extracted trace behaviour may vary across instruments
- rawR/rawrr lacks parallel computation support, which is only a planned next step for big computational proteomics in R
- On Windows, the decimal symbol must be configured as '.' for m/z and intensity vectors to have correct lengths
- rawR does not yet provide an exchangeable Spectra backend (MsBackendRawFileReader integration was only planned)

## Evidence

- [methods] runtime installation step: [evidence span withheld — anchor status: not_found]
