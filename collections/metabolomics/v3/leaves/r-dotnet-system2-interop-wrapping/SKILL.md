---
name: r-dotnet-system2-interop-wrapping
description: Use when an R workflow must read spectra, TIC/XIC chromatograms, or file
  headers directly from Thermo Fisher .raw Orbitrap files without converting them
  to mzML, and a vendor API wrapper executable is or can be built.
when_to_use_negative:
- The data is already in an exchange format such as mzML — use standard Spectra/mzR
  backends instead of the vendor API wrapper
- The goal is LC-MS method optimization metadata visualization rather than raw spectral
  access — rawDiag is the dedicated package for that
- No Mono/.NET runtime or prebuilt rawrr.exe for the platform is available and building
  it is not possible — the wrapper cannot execute
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package whose readSpectrum()/readChromatogram() wrappers call rawrr.exe
    via base::system2 and parse temp-file output into S3 objects
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Thermo Fisher vendor .NET/C# assemblies that read RAW files and handle scan-filter
    validation and application
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative MsBackend for Bioconductor Spectra built on rawrr accessor functions
  repo: https://github.com/cpanse/MsBackendRawFileReader
- name: rawDiag
  role: Proof-of-principle package showing raw-file access by wrapping vendor-provided
    APIs (metadata analysis/visualization)
  repo: https://github.com/fgcz/rawDiag
- name: tartare (ExperimentHub EH4547)
  role: Bioconductor data package supplying the example raw file for testing the interop
    path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:3c265a705eb1478a7f61b89bf6917df75f231e34ccfbfc01fb559cd11684c290
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
  - RawFileReader
  - rawDiag
  - rawR
  - rawrr
  - tartare (ExperimentHub EH4547)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# r-dotnet-system2-interop-wrapping

## Summary

A pattern for giving R direct access to spectral data in proprietary Thermo Fisher Scientific .raw files by wrapping the vendor's RawFileReader .NET assemblies (C#) behind an executable (rawrr.exe) that R invokes via base::system2, avoiding prior conversion to mzML or other exchange formats.

## When to use

Use when an R workflow must read spectra, TIC/XIC chromatograms, or file headers directly from Thermo Fisher .raw Orbitrap files without converting them to mzML, and a vendor API wrapper executable is or can be built.

## When NOT to use

- The data is already in an exchange format such as mzML — use standard Spectra/mzR backends instead of the vendor API wrapper
- The goal is LC-MS method optimization metadata visualization rather than raw spectral access — rawDiag is the dedicated package for that
- No Mono/.NET runtime or prebuilt rawrr.exe for the platform is available and building it is not possible — the wrapper cannot execute

## Inputs

- Thermo Fisher Scientific raw file (e.g., 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- rawrr.exe assembly built with 'dotnet publish rawrr-dotnet.csproj --os <os> -a x64' plus ThermoFisher.CommonCore RawFileReader .NET assemblies
- Scan numbers, filter strings, or m/z target lists with tolerance in ppm (e.g., mass = iRTmz, tol = 10, type = 'xic', filter = 'ms')

## Outputs

- rawrrSpectrum or rawrrSpectrumSet S3 objects containing numeric m/z–intensity vectors for centroided, lock-mass-corrected scans
- rawrrChromatogram S3 object for a TIC (all MS1-level scans) or XIC traces computed with 10 ppm tolerance
- File header information returned as a simple R list (readFileHeader())

## How to apply

Build the rawrr.exe assembly for the target platform with 'dotnet publish rawrr-dotnet.csproj --os <osx|win|linux> -a x64', shipping it together with the ThermoFisher.CommonCore RawFileReader assemblies; run it under the .NET 8.0 runtime on Windows or Mono on Linux/macOS. From R, call base::system2 on rawrr.exe with the raw file path and a request (e.g., a scan number or a filter string such as 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'); the C# layer calls the vendor API, which validates and applies scan filters, so R needs no a priori knowledge of which scans match. rawrr.exe writes results to a temporary location on disk; R then reads the temp output back and parses it into S3 objects — rawrrSpectrum/rawrrSpectrumSet for centroided scans and rawrrChromatogram for TIC/XIC traces. Run the package's validation functions on the constructed instances before further analysis, and rely on example data such as 20181113_010_autoQC01.raw (MD5 a1f5df9627cf9e0d51ec1906776957ab, via ExperimentHub EH4547 in the tartare package or MassIVE MSV000086542) for sanity checks.

## Related tools

- **rawrr** (R package whose readSpectrum()/readChromatogram() wrappers call rawrr.exe via base::system2 and parse temp-file output into S3 objects) — https://github.com/fgcz/rawrr
- **RawFileReader** (Thermo Fisher vendor .NET/C# assemblies that read RAW files and handle scan-filter validation and application) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative MsBackend for Bioconductor Spectra built on rawrr accessor functions) — https://github.com/cpanse/MsBackendRawFileReader
- **rawDiag** (Proof-of-principle package showing raw-file access by wrapping vendor-provided APIs (metadata analysis/visualization)) — https://github.com/fgcz/rawDiag
- **tartare (ExperimentHub EH4547)** (Bioconductor data package supplying the example raw file for testing the interop path)

## Examples

```
eh <- ExperimentHub::ExperimentHub(); EH4547 <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(EH4547, 9594)
```

## Evaluation signals

- Constructed rawrrSpectrum/rawrrChromatogram instances pass the package's validation functions
- Parsed spectrum contains numeric m/z–intensity vectors consistent with the centroided, lock-mass-corrected content of the sample file
- The TIC spans the MS1-level scans of the run (e.g., the 35 min autoQC01 run with 2.1881 × 10^4 scans)
- Example raw file integrity check: MD5 equals a1f5df9627cf9e0d51ec1906776957ab
- Scan-level cross-check against MassIVE USI assignments (e.g., scan 9594 = LGGNEQVTR, MS-GF+ Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4)

## Limitations

- Requires a .NET runtime (.NET 8.0 on Windows, Mono on Linux/macOS) and a rawrr.exe binary published per target OS/architecture (x64)
- At the initial release, no exchangeable-backend support and no parallel computation support were implemented (planned future work)
- As of the manuscript, rawR was not yet submitted to Bioconductor and alignment with the R for Mass Spectrometry initiative was still pending

## Evidence

- [abstract] rawR provides OS-independent raw access via vendor API wrapper: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [methods] Compile rawrr.exe per platform with dotnet publish: "dotnet publish rawrr-dotnet.csproj --os osx -a x64 --output /Users/cp/Library/Caches/org.R-project.R/R/rawrr/rawrrassembly/osx-x64"
- [results] Vendor API applies scan filters so R needs no filter knowledge: "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [readme] rawrr wraps RawFileReader .NET assemblies: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
