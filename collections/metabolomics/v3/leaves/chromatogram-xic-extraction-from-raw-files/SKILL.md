---
name: chromatogram-xic-extraction-from-raw-files
description: Use when the input is a Thermo Fisher Orbitap .raw file (e.g., 20181113_010_autoQC01.raw)
  and you need chromatographic traces/XICs for target masses in R, e.g., to derive
  retention times for iRT peptides.
when_to_use_negative:
- Input data has already been converted to an exchange format like mzML (the rawR
  workflow exists to avoid ProteoWizard conversion of Thermo .raw files)
- The raw file is not from a Thermo Fisher Scientific instrument — rawrr wraps only
  the Thermo RawFileReader .NET API
- The analysis operates on preprocessed/peptide-level estimates (e.g., MSstats processing
  MaxQuant or Skyline outputs) rather than raw spectral/chromatographic data
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: rawrr
  role: R package providing readChromatogram XIC extraction (10 ppm tolerance, filter
    = 'ms') directly from Thermo .raw files; requires installRawrrExe() runtime
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET (C#) API wrapped by rawrr to read chromatographic and
    spectral data from proprietary Thermo raw files on Windows, Linux, and MacOS
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Statistical environment in which XICs are extracted and downstream iRT regression
    (lm()) is performed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:e183ca4a3c78ccb6454e1516d4fd5a4046e7a2feaea00a773767729878eb7a84
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# chromatogram-xic-extraction-from-raw-files

## Summary

Extracts MS1-level extracted ion chromatograms (XICs) directly from proprietary Thermo Fisher .raw files in R using rawrr::readChromatogram, enabling downstream analyses such as iRT retention-time regression without converting raw data to mzML via ProteoWizard.

## When to use

Use when the input is a Thermo Fisher Orbitap .raw file (e.g., 20181113_010_autoQC01.raw) and you need chromatographic traces/XICs for target masses in R, e.g., to derive retention times for iRT peptides.

## When NOT to use

- Input data has already been converted to an exchange format like mzML (the rawR workflow exists to avoid ProteoWizard conversion of Thermo .raw files)
- The raw file is not from a Thermo Fisher Scientific instrument — rawrr wraps only the Thermo RawFileReader .NET API
- The analysis operates on preprocessed/peptide-level estimates (e.g., MSstats processing MaxQuant or Skyline outputs) rather than raw spectral/chromatographic data

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file (e.g., 20181113_010_autoQC01.raw from MSV000086542 / ExperimentHub EH4547)
- List of target monoisotopic masses (e.g., iRT peptide m/z values)
- Chromatogram filter string and mass tolerance (10 ppm, MS1-level filter = 'ms')

## Outputs

- Extracted ion chromatogram objects (mZ and intensity traces per target mass) from rawrr::readChromatogram
- Per-peptide retention times at maximum intensity
- Plots of RT vs iRT score with fitted regression line; report (irt_regression_report.txt) with R-squared, slope, and intercept

## How to apply

Fetch the example raw file 20181113_010_autoQC01.raw via ExperimentHub (EH4547, MassIVE MSV000086542) and first install the .NET runtime with rawrr::installRawrrExe(), since rawrr wraps the vendor RawFileReader API. For each target mass (e.g., iRT peptide monoisotopic masses), call rawrr::readChromatogram with a tolerance of 10 ppm around the target mass and restrict to MS1-level scans using filter = 'ms'. For each peptide, determine the retention time at the maximum of the intensity trace. These RTs then feed downstream steps such as fitting rt = a + b * score with lm(), where the slope (~1.5 %B/min gradient change rate) and intercept (predicted RT of GAGSSEPVTGLDAK, the zero-score iRT peptide) validate the extraction.

## Related tools

- **rawrr** (R package providing readChromatogram XIC extraction (10 ppm tolerance, filter = 'ms') directly from Thermo .raw files; requires installRawrrExe() runtime) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET (C#) API wrapped by rawrr to read chromatographic and spectral data from proprietary Thermo raw files on Windows, Linux, and MacOS) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Statistical environment in which XICs are extracted and downstream iRT regression (lm()) is performed) — https://www.r-project.org/

## Examples

```
chr <- rawrr::readChromatogram(rawfile, masses, tol = 10, filter = "ms")
```

## Evaluation signals

- Each XIC must have been computed with a 10 ppm tolerance around the target mass using only MS1-level scans (filter = 'ms')
- Retention time per peptide corresponds to the maximum of the extracted intensity trace
- Downstream iRT fit shows high linearity (high R-squared)
- Fitted slope is consistent with the ~1.5 %B/min gradient change rate
- Fitted intercept equals the predicted RT of the zero-score iRT peptide GAGSSEPVTGLDAK

## Limitations

- On Windows, the decimal symbol has to be configured as a '.' or mZ and intensity vectors may not have correct lengths
- rawrrSpectrum/chromatogram content partially depends on the instrument model and installed instrument control software version
- The rawrr.exe .NET runtime must be installed first via rawrr::installRawrrExe(); parallel computation support is not yet present

## Evidence

- [results] Each XIC was calculated using a tolerance of 10 ppm around the target mass and using only MS1-level scans.: "Each XIC was calculated using a tolerance of 10 ppm around the target mass and using only MS1-level scans."
- [abstract] rawR provides OS-independent access to Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files"
- [methods] rawrr runtime installation prerequisite: [evidence span withheld — anchor status: not_found]
- [readme] README: rawrr purpose: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package"
