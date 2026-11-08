---
name: peptide-retention-time-peak-picking
description: Use when the input is a Thermo Fisher Orbitrap .raw file (e.g., 20181113_010_autoQC01.raw)
  containing iRT peptide runs and the goal is calibration-grade retention times for
  iRT regression, i.e., fitting rt = a + b * iRT-score.
when_to_use_negative:
- Input data has already been converted to mzML or another open exchange format and
  the raw file itself is unavailable — this skill relies on direct readChromatogram
  access to Thermo .raw files via RawFileReader.
- The goal is MS2-level fragment analysis (e.g., spectrum interpretation of scan 9594
  / LGGNEQVTR) rather than MS1-based retention-time extraction for iRT calibration.
- The peptides of interest lack assigned iRT scale scores, so no rt = a + b * score
  regression can be fit.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rawrr
  role: Provides readChromatogram() for computing XICs from the raw file with a 10
    ppm tolerance and filter = 'ms', and installRawrrExe() to set up the rawrr.exe
    runtime
  repo: https://github.com/fgcz/rawrr
- name: R
  role: Statistical environment used for fitting the iRT regression with lm() and
    for plotting RT vs. iRT values
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr to access spectral/chromatographic
    data logged in the proprietary Thermo Fisher .raw file
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:cdce80735c035dbc17782b885e1e23b8be84f754f260c555fd03c0afd26cc852
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# peptide-retention-time-peak-picking

## Summary

Extracts retention times for iRT (or indexed retention-time) peptides by computing extracted ion chromatograms (XICs) directly from Thermo Fisher .raw files via rawrr::readChromatogram, picking the RT at the maximum of each intensity trace, and fitting a linear iRT regression model rt = a + b * score with R's lm().

## When to use

Use when the input is a Thermo Fisher Orbitrap .raw file (e.g., 20181113_010_autoQC01.raw) containing iRT peptide runs and the goal is calibration-grade retention times for iRT regression, i.e., fitting rt = a + b * iRT-score.

## When NOT to use

- Input data has already been converted to mzML or another open exchange format and the raw file itself is unavailable — this skill relies on direct readChromatogram access to Thermo .raw files via RawFileReader.
- The goal is MS2-level fragment analysis (e.g., spectrum interpretation of scan 9594 / LGGNEQVTR) rather than MS1-based retention-time extraction for iRT calibration.
- The peptides of interest lack assigned iRT scale scores, so no rt = a + b * score regression can be fit.

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file (e.g., 20181113_010_autoQC01.raw from MassIVE MSV000086542 / ExperimentHub EH4547)
- List of iRT peptide target masses (monoisotopic m/z values) for XIC extraction
- iRT peptide scores (indexed retention-time scale values) for the regression

## Outputs

- Extracted ion chromatograms (intensity vs. retention time traces) per iRT peptide
- Picked retention time per iRT peptide (RT at maximum of the intensity trace)
- Fitted linear model (lm object) with slope (gradient change rate), intercept (predicted RT of GAGSSEPVTGLDAK), and R-squared
- RT vs. iRT scatter plot with fitted regression line and iRT regression report (irt_regression_report.txt)

## How to apply

Fetch the raw file (e.g., via ExperimentHub entry EH4547 from MassIVE MSV000086542) and install the rawrr runtime with rawrr::installRawrrExe(). Compute XICs for the iRT peptide target masses using rawrr::readChromatogram with a 10 ppm tolerance around the target mass and restricting to MS1-level scans (filter = 'ms'). For each iRT peptide, determine the retention time at the maximum of the intensity trace (peak picking on the XIC). Fit a linear model of iRT values against the extracted retention times using lm(), then report the regression R-squared, the slope b, and the intercept a. The slope is interpreted as the gradient change rate (expected ~1.5 %B/min for the 35 min LC-MS run) and the intercept equals the predicted RT of the iRT peptide GAGSSEPVTGLDAK, which is defined to have zero score on the iRT scale.

## Related tools

- **rawrr** (Provides readChromatogram() for computing XICs from the raw file with a 10 ppm tolerance and filter = 'ms', and installRawrrExe() to set up the rawrr.exe runtime) — https://github.com/fgcz/rawrr
- **R** (Statistical environment used for fitting the iRT regression with lm() and for plotting RT vs. iRT values)
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr to access spectral/chromatographic data logged in the proprietary Thermo Fisher .raw file) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
ch <- rawrr::readChromatogram(rawFile, iRTmz, type = "xic", filter = "ms")
```

## Evaluation signals

- The fitted linear model shows high R-squared (high linearity between iRT scores and picked RTs).
- The regression slope b is consistent with the expected 1.5 %B/min gradient change rate of the LC gradient.
- The intercept a equals the predicted retention time of the iRT peptide GAGSSEPVTGLDAK (defined iRT score of zero).
- XICs were computed with exactly a 10 ppm tolerance around the target masses using only MS1-level scans (filter = 'ms').
- RT value per peptide corresponds to the maximum of its extracted ion chromatogram intensity trace.

## Limitations

- The rawrr.exe runtime must be installed first (rawrr::installRawrrExe()); data exchange between the R and C# layers uses file I/O.
- On Windows, the decimal symbol must be configured as '.' or the m/z and intensity vectors returned by rawrr may have incorrect lengths.
- The rawrrSpectrum/rawrr content partially depends on the instrument model and installed instrument control software version.

## Evidence

- [other] Slope of the iRT regression as gradient change rate: [evidence span withheld — anchor status: not_found]
