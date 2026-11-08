---
name: raw-file-scan-extraction
description: Use when the analysis goal is per-scan inspection of Thermo Orbitrap
  raw files — e.g. building a base-peak table for all MS1-level scans, reconstructing
  a scan index with MSOrder and rtinseconds, or extracting centroided m/z–intensity
  vectors for a specific scan — in R without converting to mzML.
when_to_use_negative:
- The raw files are already converted to exchange formats like mzML and you plan to
  use an mzML-based reader — rawrr reads the vendor format directly, so conversion
  pipelines are unnecessary
- The task is LC-MS method optimization / metadata visualization rather than per-scan
  spectral extraction — the rawDiag package is the article's stated tool for meta
  data analysis and visualization
- You need an exchangeable backend or parallel computation for data access — the article
  lists these as planned future work for rawR, not current capabilities
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: R package providing readSpectrum()/readIndex()/readChromatogram() wrappers
    that call the vendor RawFileReader API via rawrr.exe to extract scan-level spectral
    data and attributes from Thermo raw files
  repo: https://github.com/fgcz/rawrr
- name: Thermo Fisher RawFileReader
  role: Vendor .NET assemblies wrapped by rawrr.exe; perform scan filtering (filter
    validity checks and application) and read header, chromatographic, or spectral
    data from the RAW file
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative Spectra MsBackend built on rawrr accessor functions, for on-disk
    access to spectral data in Thermo .raw files through the Bioconductor Spectra
    interface
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:8d2fd05025fb38df9e03d9c71ae38fcfddbe964f423a5bffed0356d6ebbf4cce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - .NET
  - Bioconductor
  - Mono
  - MsBackendRawFileReader
  - R
  - Thermo Fisher RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# raw-file-scan-extraction

## Summary

Extract individual spectra and per-scan attributes (scan number, start time, base peak mass, base peak intensity) directly from proprietary Thermo Fisher Scientific .raw files in R via the rawrr/rawR package, without prior conversion to mzML.

## When to use

Use when the analysis goal is per-scan inspection of Thermo Orbitrap raw files — e.g. building a base-peak table for all MS1-level scans, reconstructing a scan index with MSOrder and rtinseconds, or extracting centroided m/z–intensity vectors for a specific scan — in R without converting to mzML.

## When NOT to use

- The raw files are already converted to exchange formats like mzML and you plan to use an mzML-based reader — rawrr reads the vendor format directly, so conversion pipelines are unnecessary
- The task is LC-MS method optimization / metadata visualization rather than per-scan spectral extraction — the rawDiag package is the article's stated tool for meta data analysis and visualization
- You need an exchangeable backend or parallel computation for data access — the article lists these as planned future work for rawR, not current capabilities

## Inputs

- Proprietary Thermo Fisher Scientific raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Scan-selection filter string (e.g. 'ms' for MS1-level scans, or a full filter like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')
- Target scan numbers or a scan index obtained via readIndex()

## Outputs

- rawrrSpectrumSet of centroided, lock-mass-corrected spectra with per-scan summary attributes (Scan Start Time, Scan Number, Base Peak Intensity, Base Peak Mass)
- Per-scan scan index with rtinseconds and MSOrder (from readIndex())
- Base-peak data frame with columns scan, StartTime, mZ, intensity (e.g. written to basepeak_ms1.csv)

## How to apply

Obtain the target raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab, from MassIVE MSV000086542 or ExperimentHub EH4547) and ensure the rawrr.exe assembly plus ThermoFisher.CommonCore RawFileReader .NET assemblies are available (.NET 8.0 runtime on Windows, Mono on Linux/macOS). Read the scan index with rawrr's readIndex() to get rtinseconds and MSOrder per scan, and use that to select the scans matching the desired MS order. Call readSpectrum() restricted to the selected scan numbers (e.g. filter 'ms' for MS1); the C# wrapper invokes the vendor RawFileReader API, which handles the scan filtering process — checking filter validity and applying the filter — so R code needs no a priori knowledge of which scans match. The returned rawrrSpectrumSet exposes, per scan, summary attributes such as 'Scan Start Time', 'Scan Number', 'Base Peak Intensity', and 'Base Peak Mass' alongside numeric m/z–intensity vectors; for a base-peak table, identify for each spectrum the peak with the maximum signal intensity and record its m/z, intensity, and the StartTime metadata key. Assemble the per-scan results into a data frame (e.g. columns scan, StartTime, mZ, intensity) and write it out (e.g. basepeak_ms1.csv); validation requires exactly one row per MS1-level scan with intensity equal to each spectrum's maximum signal intensity.

## Related tools

- **rawrr** (R package providing readSpectrum()/readIndex()/readChromatogram() wrappers that call the vendor RawFileReader API via rawrr.exe to extract scan-level spectral data and attributes from Thermo raw files) — https://github.com/fgcz/rawrr
- **Thermo Fisher RawFileReader** (Vendor .NET assemblies wrapped by rawrr.exe; perform scan filtering (filter validity checks and application) and read header, chromatographic, or spectral data from the RAW file) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative Spectra MsBackend built on rawrr accessor functions, for on-disk access to spectral data in Thermo .raw files through the Bioconductor Spectra interface) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
EH4547 <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(rawfile, scan = ms1_scans, filter = "ms")
```

## Evaluation signals

- The base-peak table contains exactly one row per MS1-level scan in the file
- Each row's intensity equals the maximum signal intensity of the corresponding spectrum
- The parsed rawrrSpectrum objects pass rawrr's validation functions on the constructed class instances
- The retrieved 'Scan Start Time' / rtinseconds values are consistent between the readIndex() scan index and the readSpectrum() summary attributes
- For known scans (e.g. scan 9594, the doubly-charged iRT peptide LGGNEQVTR), fragment ion signals are several ten or even hundred fold above the local noise estimate

## Limitations

- rawR requires the rawrr.exe assembly and ThermoFisher.CommonCore RawFileReader .NET assemblies; on Linux/macOS this depends on the Mono runtime (article notes .NET 8.0 on Windows)
- No exchangeable backend support or parallel computation for data access yet — both are listed as planned future work
- rawR was not yet submitted to the Bioconductor project at the time of the article (submission planned)
- The wrapper writes extracted information to a temporary location on the hard drive before it is read back into R

## Evidence

- [other] rawrr provides per-scan access to scan index metadata and spectral attributes: [evidence span withheld — anchor status: not_found]
- [other] readIndex provides rtinseconds and MSOrder for filtering scans by MS order: [evidence span withheld — anchor status: not_found]
- [other] vendor API handles scan filtering so R needs no prior scan knowledge: [evidence span withheld — anchor status: not_found]
- [abstract] rawR provides complete, OS independent access to all spectral data in Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [methods] Example raw file 20181113_010_autoQC01.raw from MassIVE MSV000086542: "The file is part of the MassIVE dataset [MSV000086542]"
- [methods] Spectra are centroided and lock-mass corrected on disk: "All spectra were written to disk after applying centroiding (c) and lock mass correction"
- [results] readSpectrum returns rawrrSpectrum object or rawrrSpectrumSet: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [readme] README: rawrr wraps RawFileReader .NET assemblies for OS-independent access: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
