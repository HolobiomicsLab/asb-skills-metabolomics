---
name: base-peak-identification-per-spectrum
description: Use when you have a Thermo .raw file from an Orbitrap instrument and
  need per-MS1-scan base-peak m/z and intensity values, e.g. to track signal drift
  or confirm acquisition quality across a run.
when_to_use_negative:
- When you need fragment-ion-level (MS2) peak information — rawrr's filter-based readSpectrum
  use cases in the article target MS2 scans with specific filters like 'FTMS + c NSI
  Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' rather than a base-peak-per-MS1
  summary
- When the input is already a processed feature table or exported chromatogram (e.g.
  XIC or TIC output from readChromatogram) rather than raw vendor .raw data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package wrapping the Thermo RawFileReader .NET API; its readSpectrum() function
    returns rawrrSpectrum objects whose summary exposes Base Peak Mass and Base Peak
    Intensity for MS1 scans, enabling the per-scan base-peak table construction
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET assemblies invoked by rawrr.exe to read Thermo .raw files;
    performs the low-level header, chromatographic, and spectral data extraction
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative Spectra backend that exposes rawrr accessors; relevant if the
    workflow is implemented through the Bioconductor Spectra package instead of rawrr
    directly
  repo: https://github.com/fgcz/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:4655caf088f708fd7ee5315f7484c995084498cea19459ece0f4abe211cfaf89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - Bioconductor
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# base-peak-identification-per-spectrum

## Summary

Extract the base peak mass and base peak intensity from every centroided MS1-level scan of a Thermo Fisher raw file using rawrr's readSpectrum(), producing a per-scan base-peak table (scan, retention time, m/z, intensity) for QC and chromatography sanity checks.

## When to use

Use when you have a Thermo .raw file from an Orbitrap instrument and need per-MS1-scan base-peak m/z and intensity values, e.g. to track signal drift or confirm acquisition quality across a run.

## When NOT to use

- When you need fragment-ion-level (MS2) peak information — rawrr's filter-based readSpectrum use cases in the article target MS2 scans with specific filters like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' rather than a base-peak-per-MS1 summary
- When the input is already a processed feature table or exported chromatogram (e.g. XIC or TIC output from readChromatogram) rather than raw vendor .raw data

## Inputs

- Thermo Fisher Scientific .raw file (proprietary Orbitrap format, e.g. 20181113_010_autoQC01.raw)
- MS order filter string (filter = 'ms') to select MS1-level scans

## Outputs

- Data frame / CSV (basepeak_ms1.csv) with columns: scan, StartTime, mZ, intensity — one row per MS1-level scan
- rawrrSpectrumSet of centroided MS1 spectra (intermediate object)

## How to apply

Obtain the example Thermo raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547) and call rawrr::readSpectrum() on it with filter 'ms' to restrict to MS1-level scans; this invokes the compiled rawrr.exe wrapper around the vendor RawFileReader .NET API and returns a rawrrSpectrumSet of centroided spectra. For each returned spectrum, inspect its summary metadata which exposes Scan Number, Scan Start Time, Base Peak Mass and Base Peak Intensity. Identify the peak with the maximum signal intensity and record its m/z and intensity, then assemble one row per MS1 scan into a data frame with columns scan, StartTime, mZ, intensity and write it to basepeak_ms1.csv. Correct application means exactly one row per MS1-level scan in the file, with each row's intensity equal to the maximum signal intensity of that spectrum, and retention times consistent with the file's scan index (readIndex() provides rtinseconds and MSOrder for filtering scans by MS order).

## Related tools

- **rawrr** (R package wrapping the Thermo RawFileReader .NET API; its readSpectrum() function returns rawrrSpectrum objects whose summary exposes Base Peak Mass and Base Peak Intensity for MS1 scans, enabling the per-scan base-peak table construction) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET assemblies invoked by rawrr.exe to read Thermo .raw files; performs the low-level header, chromatographic, and spectral data extraction) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative Spectra backend that exposes rawrr accessors; relevant if the workflow is implemented through the Bioconductor Spectra package instead of rawrr directly) — https://github.com/fgcz/MsBackendRawFileReader

## Examples

```
S <- rawrr::readSpectrum(rawfile, filter = "ms"); bp <- data.frame(scan = sapply(S, function(x) x$Scan$ScanNumber), StartTime = sapply(S, function(x) x$Scan$StartTime), mZ = sapply(S, function(x) x$Scan$BasePeakMass), intensity = sapply(S, function(x) x$Scan$BasePeakIntensity)); write.csv(bp, "basepeak_ms1.csv")
```

## Evaluation signals

- Output table contains exactly one row per MS1-level scan in the raw file (row count should match the MS1 scan count reported by readIndex() where MSOrder == 1)
- Each row's intensity equals the maximum signal intensity of the corresponding centroided MS1 spectrum
- Retention times (StartTime) must be strictly increasing across the table and consistent with rtinseconds values in the scan index
- mZ and intensity values for the base peak match the Base Peak Mass and Base Peak Intensity entries exposed in the rawrrSpectrum summary
- CSV schema check: columns must be exactly scan, StartTime, mZ, intensity

## Limitations

- rawrr relies on a compiled C# wrapper (rawrr.exe) that must be built for the correct OS/architecture (osx/win/linux via dotnet publish); build failures block all spectral reading
- Only spectral data logged in Thermo Fisher Scientific .raw files is accessible; this skill does not apply to mzML or other converted formats without prior conversion
- The article notes rawR was not yet submitted to Bioconductor at time of writing, with exchangeable backends and parallel computation support planned but not yet implemented

## Evidence

- [abstract] rawR provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [results] readSpectrum returns rawrrSpectrum objects: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [results] readIndex provides rtinseconds and MSOrder for filtering by MS order: [evidence span withheld — anchor status: not_found]
- [results] readChromatogram with filter='ms' shows MS1 filtering convention: "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [readme] rawrr wraps RawFileReader .NET assemblies: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
- [readme] MsBackendRawFileReader provides alternative Spectra backend: [evidence span withheld — anchor status: external_doc]
- [results] 35 min run resulted in 2.1881 x 10^4 scans written to disc: "The 35 min run resulted in 2.1881 × 104 scans that were written to disc."
