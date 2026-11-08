---
name: mass-spectrometry-chromatogram-visualization
description: Use when the input is a Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw)
  and the goal is to plot extracted ion chromatograms for target masses, inspect fragment-ion
  traces, or derive retention times for RT/iRT calibration.
when_to_use_negative:
- Input data is already converted to an exchange format like mzML — then standard
  R/Bioconductor chromatogram tooling on mzML applies instead of rawrr
- The analysis requires a Spectra-compatible on-disk backend — MsBackendRawFileReader
  is still WIP ('Initial development is in progress'), so Spectra-based pipelines
  cannot yet rely on it
- Peak picking/quantification of features across many samples is needed — this skill
  targets trace extraction and visualization from single raw files, not feature finding
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Wraps Thermo Fisher RawFileReader to read chromatograms (readChromatogram)
    and spectra directly from .raw files in R
  repo: https://github.com/fgcz/rawrr
- name: rawR
  role: OS-independent R package providing complete access to all spectral data logged
    in Thermo Fisher raw files (precursor of rawrr)
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET/C# assemblies underlying rawrr's raw-file access (header,
    chromatographic, and spectral data)
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Environment in which chromatograms are extracted, plotted, and used for iRT
    regression via lm()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:601c5b88f103b7d7e43ca2924a38d0aecbe4d65c181b50f22162be8bd937bccf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - ExperimentHub
  - R
  - RawFileReader
  - rawR
  - rawrr
  - tartare
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- base-r-chromatogram-visualization
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectrometry-chromatogram-visualization

## Summary

Extract and visualize chromatographic traces (XICs, fragment-ion traces) directly from Thermo Fisher .raw files in R using rawrr's readChromatogram, without prior file conversion. This skill supports retention-time calibration (iRT regression), peptide trace inspection, and fragment-ion (y-ion series) display.

## When to use

Use when the input is a Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw) and the goal is to plot extracted ion chromatograms for target masses, inspect fragment-ion traces, or derive retention times for RT/iRT calibration.

## When NOT to use

- Input data is already converted to an exchange format like mzML — then standard R/Bioconductor chromatogram tooling on mzML applies instead of rawrr
- The analysis requires a Spectra-compatible on-disk backend — MsBackendRawFileReader is still WIP ('Initial development is in progress'), so Spectra-based pipelines cannot yet rely on it
- Peak picking/quantification of features across many samples is needed — this skill targets trace extraction and visualization from single raw files, not feature finding

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file (e.g. 20181113_010_autoQC01.raw via ExperimentHub EH4547 / MassIVE MSV000086542)
- Target m/z values for iRT peptides or fragment ions (e.g. doubly-charged LGGNEQVTR at scan 9594, m/z 487.2567)
- Raw filter string (e.g. 'ms' for MS1-only, or 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')

## Outputs

- Extracted ion chromatograms (intensity vs retention time traces) per target mass
- Per-peptide retention times (time at maximum of the intensity trace)
- Plots: chromatogram traces and RT vs iRT scatter plot with fitted regression line
- Regression report (irt_regression_report.txt) with R-squared, slope, and intercept

## How to apply

Install the rawrr runtime with rawrr::installRawrrExe() and fetch the raw file via ExperimentHub (EH4547 from the Bioconductor 'tartare' package, originally MassIVE MSV000086542). Compute XICs with rawrr::readChromatogram using a 10 ppm tolerance around each target mass and restricting to MS1-level scans via filter = 'ms'; for fragment-ion traces specify the raw filter string such as 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'. For each trace, identify the retention time at the maximum of the intensity trace, then optionally fit rt = a + b * irt with lm() and plot RT vs iRT values together with the fitted line. The rationale is that rawrr wraps the vendor RawFileReader .NET API so chromatograms are read from the proprietary file directly, with data returned to R via file I/O, avoiding conversion to mzML with ProteoWizard.

## Related tools

- **rawrr** (Wraps Thermo Fisher RawFileReader to read chromatograms (readChromatogram) and spectra directly from .raw files in R) — https://github.com/fgcz/rawrr
- **rawR** (OS-independent R package providing complete access to all spectral data logged in Thermo Fisher raw files (precursor of rawrr)) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET/C# assemblies underlying rawrr's raw-file access (header, chromatographic, and spectral data)) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Environment in which chromatograms are extracted, plotted, and used for iRT regression via lm())

## Examples

```
rawrr::readChromatogram(rawfile, mass, tol = 10, filter = "ms")
```

## Evaluation signals

- XICs must be computed with exactly 10 ppm tolerance around the target mass and only MS1-level scans (filter = 'ms')
- The 35 min LC-MS run should yield ~2.1881 x 10^4 scans; chromatogram extraction should cover the run's retention-time range
- For iRT regression, accept only if the RT vs iRT fit shows high R-squared, slope consistent with the 1.5 %B/min gradient change rate, and intercept equal to the predicted RT of GAGSSEPVTGLDAK (zero-score iRT peptide)
- Fragment-ion trace plots should show y-ion signals several ten or even hundred folds above the noise estimate (e.g. spectrum of LGGNEQVTR/2, scan 9594, MS-GF+ Score 144)
- rawrrSpectrum content can vary with instrument model and instrument control software version — check trace attributes for consistency across files

## Limitations

- On Windows, the decimal symbol must be configured as '.' or mZ and intensity vectors returned by rawrr will have incorrect lengths
- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- rawrr.exe is a cross-compiled .NET runtime (dotnet publish for osx/win/linux x64) that must be installed via rawrr::installRawrrExe() before any chromatogram access

## Evidence

- [methods] runtime installation step: [evidence span withheld — anchor status: not_found]
