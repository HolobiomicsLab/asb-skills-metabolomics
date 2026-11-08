---
name: irt-linear-regression-modeling
description: Use when a Thermo Fisher .raw file (e.g., a PRM autoQC run like 20181113_010_autoQC01.raw)
  contains iRT peptide chromatogram traces and you need an RT-to-iRT regression, its
  R-squared, slope, and intercept for retention-time calibration.
when_to_use_negative:
- The input has already been converted to an exchange format like mzML via ProteoWizard/MSconvert
  — rawrr reads proprietary Thermo .raw files directly via RawFileReader.
- The raw file is from a non-Thermo vendor or lacks detectable iRT peptide MS1 signals,
  so no XIC maxima can be assigned to anchor the regression.
- The goal is peptide identification or scoring (e.g., MS-GF+ assignment of spectra
  such as scan 9594 to LGGNEQVTR) rather than retention-time calibration.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: Installs the runtime (installRawrrExe) and computes extracted ion chromatograms
    for iRT peptide masses via readChromatogram with 10 ppm tolerance and filter =
    'ms'
  repo: https://github.com/fgcz/rawR
- name: R
  role: Statistical environment in which the iRT vs. retention-time linear model is
    fitted with lm() and the fitted line is plotted
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr/rawrr.exe to access chromatographic
    and spectral data in the proprietary raw file
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:35f9586a27daa5ae218abb05a01dddfd68821b0f887587506db42812e77dbb68
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# irt-linear-regression-modeling

## Summary

Calibrates retention-time prediction by fitting a linear model rt = a + b * iRT score to retention times extracted from iRT peptide chromatogram traces in a Thermo raw file, using rawrr XIC extraction at 10 ppm tolerance on MS1 scans. The fitted slope corresponds to the gradient change rate (~1.5 %B/min) and the intercept predicts the RT of the zero-score iRT peptide GAGSSEPVTGLDAK.

## When to use

Use when a Thermo Fisher .raw file (e.g., a PRM autoQC run like 20181113_010_autoQC01.raw) contains iRT peptide chromatogram traces and you need an RT-to-iRT regression, its R-squared, slope, and intercept for retention-time calibration.

## When NOT to use

- The input has already been converted to an exchange format like mzML via ProteoWizard/MSconvert — rawrr reads proprietary Thermo .raw files directly via RawFileReader.
- The raw file is from a non-Thermo vendor or lacks detectable iRT peptide MS1 signals, so no XIC maxima can be assigned to anchor the regression.
- The goal is peptide identification or scoring (e.g., MS-GF+ assignment of spectra such as scan 9594 to LGGNEQVTR) rather than retention-time calibration.

## Inputs

- Thermo Fisher Scientific .raw file (e.g., 20181113_010_autoQC01.raw from MassIVE MSV000086542 / ExperimentHub EH4547)
- Table of iRT peptides with monoisotopic masses and iRT scores (including GAGSSEPVTGLDAK with score 0)

## Outputs

- irt_regression_report.txt containing regression R-squared, slope (~1.5 %B/min), and intercept (predicted RT of GAGSSEPVTGLDAK)
- Plot of RT versus iRT values with the fitted regression line
- Fitted lm() model object rt ~ irt

## How to apply

Fetch the example raw file 20181113_010_autoQC01.raw via ExperimentHub (EH4547, MassIVE MSV000086542) and install the rawrr runtime with rawrr::installRawrrExe(). Compute extracted ion chromatograms for the iRT peptide masses using rawrr::readChromatogram with a tolerance of 10 ppm around each target mass, restricted to MS1-level scans via filter = 'ms'. For each iRT peptide, take the retention time at the maximum of the intensity trace. Fit a linear model of iRT values against the extracted retention times with R's lm(), yielding slope b (equivalent of the gradient change rate, expected ~1.5 %B/min) and intercept a (the predicted RT of GAGSSEPVTGLDAK, defined to have zero iRT score). Report R-squared, slope, and intercept to irt_regression_report.txt and plot RT versus iRT values with the fitted line; accept the fit only if R-squared is high, the slope matches the gradient rate, and the intercept equals the zero-score peptide's predicted RT.

## Related tools

- **rawrr** (Installs the runtime (installRawrrExe) and computes extracted ion chromatograms for iRT peptide masses via readChromatogram with 10 ppm tolerance and filter = 'ms') — https://github.com/fgcz/rawR
- **R** (Statistical environment in which the iRT vs. retention-time linear model is fitted with lm() and the fitted line is plotted)
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr/rawrr.exe to access chromatographic and spectral data in the proprietary raw file) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
xc <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, tolUnit = "ppm", filter = "ms"); fit <- lm(rt ~ irt, data = irtTable)
```

## Evaluation signals

- Regression R-squared is high, confirming strong linearity between iRT scores and extracted retention times.
- Slope b is consistent with the 1.5 %B/min gradient change rate of the LC method.
- Intercept a equals the predicted RT of the iRT peptide GAGSSEPVTGLDAK, whose iRT score is defined as zero.
- XICs were extracted with the documented parameters: 10 ppm tolerance around the target mass and MS1-level scans only (filter = 'ms').
- Report file irt_regression_report.txt and the RT-vs-iRT plot with the fitted line are produced as required.

## Limitations

- rawrr requires the rawrr.exe runtime; on Windows the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths.
- Exchangeable Spectra backends (MsBackendRawFileReader) were only planned at time of writing, so direct Spectra-based workflows are not available.
- rawrrSpectrum/Chromatogram content partially depends on the instrument model and installed instrument control software version.

## Evidence
