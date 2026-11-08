---
name: spectral-metadata-parsing-starttime
description: Use when you need per-scan metadata such as retention times, base peak
  m/z/intensity, or MS order from a Thermo Fisher Orbitrap .raw file to build per-scan
  tables, filter scans, or assemble XIC/TIC summaries in R.
when_to_use_negative:
- Data is already in open interchange format (mzML/mgf) parsed by a generic spectral
  backend — use Spectra/MsBackendRawFileReader or another parser instead
- The goal is chromatogram-level extraction (TIC/XIC) rather than per-scan metadata
  — use rawrr::readChromatogram() with type 'tic' or 'xic'
- Only file-level header information (no per-scan attributes) is needed — use rawrr::readFileHeader()
  instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package that reads per-scan metadata and spectra from Thermo raw files via
    readSpectrum() and readIndex()
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET API invoked by rawrr's compiled wrapper to access spectral
    data logged in .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative Spectra backend built on rawrr for direct access to spectral data
    in .raw files
  repo: https://github.com/cpanse/MsBackendRawFileReader
- name: rawDiag
  role: Related R package focused on metadata analysis and visualization of raw files
  repo: https://github.com/fgcz/rawDiag
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:e2e3b285cab5c133c811f9b7c7f3264dec509317e05afb8bd2549b8ea52cd63e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - Bioconductor
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - rawDiag
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# spectral-metadata-parsing-starttime

## Summary

Extract per-scan metadata (Scan Number, Scan Start Time, MSOrder, Base Peak Mass/Intensity) from Thermo raw files in R using the rawrr package, enabling construction of base-peak tables and MS1-filtered scan indices.

## When to use

Use when you need per-scan metadata such as retention times, base peak m/z/intensity, or MS order from a Thermo Fisher Orbitrap .raw file to build per-scan tables, filter scans, or assemble XIC/TIC summaries in R.

## When NOT to use

- Data is already in open interchange format (mzML/mgf) parsed by a generic spectral backend — use Spectra/MsBackendRawFileReader or another parser instead
- The goal is chromatogram-level extraction (TIC/XIC) rather than per-scan metadata — use rawrr::readChromatogram() with type 'tic' or 'xic'
- Only file-level header information (no per-scan attributes) is needed — use rawrr::readFileHeader() instead

## Inputs

- Thermo Fisher Scientific raw file (e.g., 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- MD5 checksum for the raw file (a1f5df9627cf9e0d51ec1906776957ab)
- Installed rawrr package with compiled rawrr.exe / .NET RawFileReader wrapper

## Outputs

- Data frame of per-scan MS1 base-peak metadata with columns scan, StartTime, mZ, intensity (e.g., basepeak_ms1.csv)
- rawrrSpectrum objects or rawrrSpectrumSet with per-scan spectral attributes
- Scan index with rtinseconds and MSOrder from readIndex()

## How to apply

Obtain a Thermo raw file (e.g., 20181113_010_autoQC01.raw via ExperimentHub EH4547 or MassIVE MSV000086542) and verify its MD5 checksum. Call rawrr::readSpectrum() restricted to MS1-level scans using the filter 'ms', which invokes the compiled C# wrapper (rawrr.exe / Thermo RawFileReader) and returns a rawrrSpectrumSet. From each rawrrSpectrum, pull its summary entries for Scan Start Time, Scan Number, Base Peak Intensity, and Base Peak Mass; optionally use rawrr::readIndex() to obtain rtinseconds and MSOrder for filtering scans by MS order. Assemble these into a data frame (e.g., columns scan, StartTime, mZ, intensity) and validate that each row corresponds to exactly one MS1-level scan whose intensity equals that spectrum's maximum signal intensity.

## Related tools

- **rawrr** (R package that reads per-scan metadata and spectra from Thermo raw files via readSpectrum() and readIndex()) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET API invoked by rawrr's compiled wrapper to access spectral data logged in .raw files) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative Spectra backend built on rawrr for direct access to spectral data in .raw files) — https://github.com/cpanse/MsBackendRawFileReader
- **rawDiag** (Related R package focused on metadata analysis and visualization of raw files) — https://github.com/fgcz/rawDiag

## Examples

```
rawfile <- normalizePath(ExperimentHub::ExperimentHub()[["EH4547"]]); s <- rawrr::readSpectrum(rawfile, filter = 'ms')
```

## Evaluation signals

- The resulting base-peak table contains exactly one row per MS1-level scan in the raw file
- Each row's intensity equals the maximum signal intensity of the corresponding rawrrSpectrum
- StartTime values match Scan Start Time entries in each rawrrSpectrum summary and agree with readIndex() rtinseconds
- MSOrder filter correctly restricts output to MS1-level scans (filter = 'ms')
- File checksum matches the published value (e.g., a1f5df9627cf9e0d51ec1906776957ab for the example raw file)

## Limitations

- rawrr depends on a compiled .NET wrapper (rawrr.exe / Thermo RawFileReader), which must be built per OS (osx/win/linux) via 'dotnet publish'
- rawR was not yet submitted to Bioconductor at time of publication; Bioconductor integration planned for the future
- No support for exchangeable backends or parallel computation at the time of writing (both listed as planned future work)
- Backed by the Thermo Fisher Scientific RawFileReader API only — cannot read other vendor formats

## Evidence

- [results] readSpectrum returns rawrrSpectrum objects with per-scan summary keys for start time, scan number, base peak intensity and mass: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [methods] The example raw file is fetched from ExperimentHub (EH4547) as part of the rawrr workflow: "eh <- ExperimentHub::ExperimentHub()
EH4547 <- normalizePath(eh[["EH4547"]])"
- [readme] readSpectrum wraps the .NET RawFileReader assembly for cross-OS access to proprietary Thermo raw files: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package or serves as MsRawFileReaderBackend for the Bioconductor Spectra package."
- [abstract] rawR provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
