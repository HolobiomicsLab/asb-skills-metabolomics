---
name: ms1-scan-filtering
description: Use when the goal is to restrict spectral access to MS1-level scans of
  a Thermo Fisher .raw file (e.g. building a base-peak table, TIC, or XIC) and the
  input is a proprietary Orbitrap raw file readable by rawR.
when_to_use_negative:
- The analysis needs fragment-ion spectra (MS2/HCD scans); the 'ms' filter excludes
  them, so use a scan filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00
  [100.0000-1015.0000]' instead.
- The raw file is not from Thermo Fisher Scientific or lacks the .raw format that
  rawR/RawFileReader can read.
- Input data are already in an open format such as mzML handled by other backends
  and direct proprietary raw access is unnecessary.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rawR/rawrr
  role: Provides the R functions readSpectrum(), readIndex() and readChromatogram()
    whose 'filter' argument (e.g. 'ms') restricts retrieval to MS1-level scans of
    the Thermo raw file.
  repo: https://github.com/fgcz/rawR
- name: Thermo RawFileReader
  role: Vendor-provided .NET API that rawrr wraps; the compiled rawrr.exe C# wrapper
    actually applies the MS1 scan filter when reading the raw file.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Bioconductor Spectra backend that can serve the same filtered raw-file access
    through Spectra accessor functions.
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:a06231b640b3fc0ab9ad72becd18cd15c49081a66ea91d7ee18342e4c30d7151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawR
  tools:
  - Bioconductor
  - MsBackendRawFileReader
  - R
  - Thermo RawFileReader
  - rawR
  - rawR/rawrr
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# MS1 scan filtering

## Summary

Filters Thermo raw file scans to the MS1 (survey) level by passing filter='ms' to rawrr's reader functions, enabling per-scan extraction of base peak m/z, base peak intensity, and scan start time for QC tables such as basepeak_ms1.csv.

## When to use

Use when the goal is to restrict spectral access to MS1-level scans of a Thermo Fisher .raw file (e.g. building a base-peak table, TIC, or XIC) and the input is a proprietary Orbitrap raw file readable by rawR.

## When NOT to use

- The analysis needs fragment-ion spectra (MS2/HCD scans); the 'ms' filter excludes them, so use a scan filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' instead.
- The raw file is not from Thermo Fisher Scientific or lacks the .raw format that rawR/RawFileReader can read.
- Input data are already in an open format such as mzML handled by other backends and direct proprietary raw access is unnecessary.

## Inputs

- Thermo Fisher Scientific proprietary .raw file (e.g. 20181113_010_autoQC01.raw from MSV000086542 or ExperimentHub EH4547)
- Scan-level filter string, e.g. 'ms', selecting MS1-order scans
- Optional scan index from rawrr::readIndex() containing rtinseconds and MSOrder per scan

## Outputs

- rawrrSpectrum or rawrrSpectrumSet containing only MS1-level centroided scans
- Data frame with columns scan, StartTime, mZ, intensity (one row per MS1 scan)
- CSV file such as basepeak_ms1.csv
- Optional chromatogram object (TIC or XIC) computed with filter='ms' via readChromatogram()

## How to apply

Acquire the example Thermo raw file (e.g. 20181113_010_autoQC01.raw from MassIVE dataset MSV000086542 or Bioconductor ExperimentHub resource EH4547) and confirm its MD5 checksum a1f5df9627cf9e0d51ec1906776957ab. Call rawrr::readSpectrum() on the raw file with the scan filter set to 'ms' so that only MS1-level scans are returned as a rawrrSpectrumSet; the call invokes the compiled C# wrapper (rawrr.exe / Thermo RawFileReader) that does the filtering. For each centroided MS1 spectrum, take the peak with the maximum signal intensity and record its m/z and intensity, and read the 'Scan Start Time' metadata key. Optionally consult readIndex(), which supplies rtinseconds and MSOrder per scan, to cross-check which scans are MS order. Assemble one row per MS1 scan into a data frame with columns scan, StartTime, mZ, intensity and write basepeak_ms1.csv; the same filter='ms' pattern applies to readChromatogram() when computing TIC/XIC traces from survey scans only.

## Related tools

- **rawR/rawrr** (Provides the R functions readSpectrum(), readIndex() and readChromatogram() whose 'filter' argument (e.g. 'ms') restricts retrieval to MS1-level scans of the Thermo raw file.) — https://github.com/fgcz/rawR
- **Thermo RawFileReader** (Vendor-provided .NET API that rawrr wraps; the compiled rawrr.exe C# wrapper actually applies the MS1 scan filter when reading the raw file.) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Bioconductor Spectra backend that can serve the same filtered raw-file access through Spectra accessor functions.) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
s <- rawrr::readSpectrum(rawfile, filter = "ms"); bp <- data.frame(scan = s$scan, StartTime = s$`Scan Start Time`, mZ = s$`Base Peak Mass`, intensity = s$`Base Peak Intensity`); write.csv(bp, "basepeak_ms1.csv", row.names = FALSE)
```

## Evaluation signals

- The result table contains exactly one row per MS1-level scan in the raw file (no MS2 rows slip through).
- Each row's intensity equals the maximum signal intensity of the corresponding MS1 spectrum.
- Each row's mZ corresponds to the base peak mass reported in the spectrum summary.
- StartTime values match the 'Scan Start Time' metadata and are consistent with rtinseconds from readIndex().
- The checksum of the input raw file matches a1f5df9627cf9e0d51ec1906776957ab before processing.

## Limitations

- rawR currently lacks exchangeable backend support and parallel computation, planned for future work, so large files may process slowly.
- The C# wrapper rawrr.exe must be compiled with dotnet publish for the target OS (osx/win/linux) before filtering works.
- Filtering relies on the filter string ('ms' or a full MS2 filter string); wrong filter syntax yields empty or wrong scan sets.

## Evidence

- [results] readSpectrum() returns rawrrSpectrum objects whose summary exposes Scan Start Time, Scan Number, Base Peak Intensity and Base Peak Mass for an individual scan: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [results] readIndex() provides rtinseconds and MSOrder for filtering scans by MS order: [evidence span withheld — anchor status: not_found]
- [results] XIC extraction with 10 ppm tolerance using only MS1-level scans via filter='ms': "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [abstract] rawR provides OS independent access to all spectral data in Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [readme] rawrr wraps RawFileReader .NET assemblies to read Thermo RAW files cross-platform: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
