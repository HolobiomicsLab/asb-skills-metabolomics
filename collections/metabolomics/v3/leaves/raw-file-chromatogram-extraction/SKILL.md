---
name: raw-file-chromatogram-extraction
description: Use when the input is a Thermo Fisher .raw file (e.g. 20181113_010_autoQC01.raw)
  and target XIC traces or full TIC/BPC are needed directly in R without converting
  to mzML via ProteoWizard.
when_to_use_negative:
- The file is already converted to mzML or another exchange format and a Spectra/MsBackendRawFileReader
  or generic parser would suffice — rawrr's value is direct .raw access.
- You need vendor-independent portability beyond the platforms rawrr.exe supports
  (Windows/macOS/Linux x64) or parallelized large-scale extraction, which the article
  lists as a planned next step.
- On Windows the system decimal symbol is not '.' — mZ and intensity vector lengths
  will be incorrect, so this skill must not be applied before fixing the locale.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: 'Primary R wrapper: readChromatogram() extracts XIC/TIC/BPC traces directly
    from the .raw file via its bundled rawrr.exe built on RawFileReader.'
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor .NET assemblies (C#) that rawrr wraps to read header, chromatographic,
    and spectral data from Thermo .raw files on Windows, Linux, and macOS.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Execution environment; the returned rawrrChromatogramSet is a native R S3
    object that can be plotted, regressed (e.g. lm() on iRT RTs), and exported.
- name: tartare
  role: Bioconductor data package supplying the example raw file EH4547 (20181113_010_autoQC01.raw)
    for demonstration and extended testing.
- name: ExperimentHub
  role: Used to fetch the example raw file EH4547 referenced in the article's R workflow.
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:cf08298a75a043fcd65f128d369c957c12c7b5f106fe1f07a615f87a49046370
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# raw-file-chromatogram-extraction

## Summary

Extracts chromatographic traces (XICs, TIC, BPC) directly from Thermo Fisher Scientific Orbitrap .raw files in R by calling the rawrr package, which wraps the vendor RawFileReader .NET API, without prior mzML conversion. Verified against the iRT peptide traces in the FGCZ system-suitability file 20181113_010_autoQC01.raw (MassIVE MSV000086542).

## When to use

Use when the input is a Thermo Fisher .raw file (e.g. 20181113_010_autoQC01.raw) and target XIC traces or full TIC/BPC are needed directly in R without converting to mzML via ProteoWizard.

## When NOT to use

- The file is already converted to mzML or another exchange format and a Spectra/MsBackendRawFileReader or generic parser would suffice — rawrr's value is direct .raw access.
- You need vendor-independent portability beyond the platforms rawrr.exe supports (Windows/macOS/Linux x64) or parallelized large-scale extraction, which the article lists as a planned next step.
- On Windows the system decimal symbol is not '.' — mZ and intensity vector lengths will be incorrect, so this skill must not be applied before fixing the locale.

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file (e.g. 20181113_010_autoQC01.raw, MassIVE MSV000086542 / ExperimentHub EH4547), MD5 a1f5df9627cf9e0d51ec1906776957ab
- Vector of target m/z values for XIC extraction (e.g. iRT peptide precursor masses)
- Optional binary filter string (e.g. "FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]")
- Installed rawrr executable runtime (rawrr::installRawrrExe())

## Outputs

- rawrrChromatogramSet object (list of rawrrChromatogram instances with m/z, intensity, time vectors)
- Per-trace summary table (e.g. apex retention times, peak areas) written as CSV such as irt_regression_summary.csv
- Validation report confirming class membership (is.rawrrChromatogram) e.g. validation_summary.txt

## How to apply

Install the rawrr runtime with rawrr::installRawrrExe() so the wrapped RawFileReader .NET executable is available, then call rawrr::readChromatogram(rawfile, mass, tol = 10, tolppm = TRUE, filter = "ms", type = "xic") for each target m/z; the 10 ppm mass tolerance and the MS1-level filter ('ms') are the parameters the article specifies for computing XICs around a target mass. For total ion or base peak chromatograms use the same readChromatogram call with type = 'tic' or 'bpc' instead of 'xic'; a binary-filter string such as "FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]" restricts fragment-ion traces to a specific MS2 scan event. The function returns a rawrrChromatogramSet (S3 class), which should be validated with is.rawrrChromatogram / validate_rawrrSpectrum-style checks before downstream use, matching the object-integrity validation the article demonstrates on scan 9594 of the autoQC01 file. Because data exchange between the R and C# layers uses file I/O, keep intermediate files writable and, on Windows, ensure the decimal symbol is '.' so m/z and intensity vectors have correct lengths. Verify output by comparing trace shape and apex RT against known iRT peptides (e.g. LGGNEQVTR) and, for the autoQC01 run, confirming the 35 min run yields ~2.1881 × 10^4 scans underlying the traces.

## Related tools

- **rawrr** (Primary R wrapper: readChromatogram() extracts XIC/TIC/BPC traces directly from the .raw file via its bundled rawrr.exe built on RawFileReader.) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor .NET assemblies (C#) that rawrr wraps to read header, chromatographic, and spectral data from Thermo .raw files on Windows, Linux, and macOS.) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Execution environment; the returned rawrrChromatogramSet is a native R S3 object that can be plotted, regressed (e.g. lm() on iRT RTs), and exported.)
- **tartare** (Bioconductor data package supplying the example raw file EH4547 (20181113_010_autoQC01.raw) for demonstration and extended testing.)
- **ExperimentHub** (Used to fetch the example raw file EH4547 referenced in the article's R workflow.)

## Examples

```
library(rawrr); rawfile <- normalizePath(ExperimentHub::ExperimentHub()[["EH4547"]]); rawrr::installRawrrExe(); xic <- rawrr::readChromatogram(rawfile, mass = c(487.2567), tol = 10, tolppm = TRUE, filter = "ms", type = "xic")
```

## Evaluation signals

- The returned object passes class checks: is.rawrrChromatogram(rawrrChromatogramSet[[i]]) is TRUE for every trace and the set validates without errors.
- For XICs of iRT peptides in 20181113_010_autoQC01.raw, extracted apex retention times reproduce the reported linear iRT regression (high R², slope/intercept consistent with published system-suitability values).
- Trace m/z vectors respect the 10 ppm tolerance: all points in an XIC lie within ±10 ppm of the target mass, and only MS1 scans (filter = 'ms') contribute when the MS1 filter is requested.
- The 35 min autoQC01 run underpinning the traces contains ~2.1881 × 10^4 scans, so trace time ranges cover the full acquisition window (~0–35 min).
- Output CSV (e.g. irt_regression_summary.csv) has one row per raw file with slope, intercept, and R² columns, and MD5 of the source .raw matches a1f5df9627cf9e0d51ec1906776957ab.

## Limitations

- rawrrSpectrum/rawrrChromatogram content partially depends on the instrument model and installed instrument control software version.
- On Windows the decimal symbol must be configured as '.'; otherwise mZ and intensity vectors have incorrect lengths.
- Parallel computation and exchangeable Spectra backends (e.g. MsBackendRawFileReader integration) are planned next steps, not fully implemented.
- Only Thermo Fisher Scientific raw files are supported; other vendor formats are out of scope.

## Evidence
