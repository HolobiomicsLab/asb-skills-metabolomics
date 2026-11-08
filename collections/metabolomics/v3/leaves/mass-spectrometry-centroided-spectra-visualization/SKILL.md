---
name: mass-spectrometry-centroided-spectra-visualization
description: Use when the input is a Thermo Fisher Scientific raw Orbitrap file and
  you need to inspect or plot centroided MS/MS scans, a total ion chromatogram over
  MS1 scans, or extracted ion chromatograms directly from R.
when_to_use_negative:
- The data is already in an exchange format such as mzML — the point of this skill
  is direct access to proprietary .raw files without conversion
- You need parallel computation or an exchangeable MsBackend — the article notes rawR
  does not yet support parallel computation or exchangeable backends for data access
  (planned future work)
- Input is not a Thermo Fisher Scientific raw file (e.g. other vendor formats) — rawR
  is specific to Thermo raw files via the RawFileReader API
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawR/rawrr
  role: R wrapper that calls rawrr.exe via base::system2 to read spectra and chromatograms
    from Thermo .raw files into rawrrSpectrum and rawrrChromatogram S3 objects
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET assemblies (C#) whose API performs scan filtering and
    reads header, chromatographic, and spectral data from the RAW file
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: rawDiag
  role: Proof-of-principle R package for wrapping vendor APIs to access raw files,
    focused on metadata analysis and LC-MS method optimization visualization
  repo: https://github.com/fgcz/rawDiag
- name: MsBackendRawFileReader
  role: Alternative MsBackend for Bioconductor Spectra providing direct access to
    spectral data in Thermo .raw files through rawrr accessor functions
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:c0ff15bb94dd086872127e30ed88e47d3185fad761427a280aaf4fea5818c0be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawR
  tools:
  - .NET
  - Bioconductor
  - Mono
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - rawDiag
  - rawR
  - rawR/rawrr
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# mass-spectrometry-centroided-spectra-visualization

## Summary

Read centroided, lock-mass-corrected spectra directly from proprietary Thermo Fisher .raw Orbitrap files into R and visualize them (individual scans, scan sets, and TIC/XIC chromatograms) without prior mzML conversion, using the rawrr package's readSpectrum/readChromatogram wrappers around the vendor RawFileReader API.

## When to use

Use when the input is a Thermo Fisher Scientific raw Orbitrap file and you need to inspect or plot centroided MS/MS scans, a total ion chromatogram over MS1 scans, or extracted ion chromatograms directly from R.

## When NOT to use

- The data is already in an exchange format such as mzML — the point of this skill is direct access to proprietary .raw files without conversion
- You need parallel computation or an exchangeable MsBackend — the article notes rawR does not yet support parallel computation or exchangeable backends for data access (planned future work)
- Input is not a Thermo Fisher Scientific raw file (e.g. other vendor formats) — rawR is specific to Thermo raw files via the RawFileReader API

## Inputs

- Thermo Fisher Scientific proprietary .raw Orbitrap file (e.g. 20181113_010_autoQC01.raw from MSV000086542 or tartare EH4547)
- rawrr.exe assembly plus ThermoFisher.CommonCore RawFileReader .NET assemblies
- Optional: vector of target m/z values (e.g. iRT peptide precursors) and mass tolerance in ppm for XIC extraction
- Optional: Thermo scan filter string (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')

## Outputs

- rawrrSpectrum / rawrrSpectrumSet S3 objects with numeric m/z and intensity vectors for centroided scans
- rawrrChromatogram objects for the total ion chromatogram (all MS1-level scans) or 10 ppm extracted ion chromatograms
- raw file header list from readFileHeader()
- Plots of centroided spectra and chromatograms via base print/plot generics

## How to apply

Obtain the raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab, from MassIVE MSV000086542 or via ExperimentHub EH4547 from the tartare data package) and ensure rawrr.exe — built with 'dotnet publish rawrr-dotnet.csproj --os <os> -a x64' for the target platform — plus the ThermoFisher.CommonCore RawFileReader assemblies are present (.NET 8.0 on Windows, Mono on Linux/macOS). Optionally inspect run metadata first with readFileHeader(), which returns a simple R list. Call readSpectrum(rawfile, scan = ...) to pull individual scans or scan collections as rawrrSpectrum / rawrrSpectrumSet objects containing numeric m/z–intensity vectors of the centroided (c) and lock-mass-corrected content, then plot with print/plot generics (e.g. scan 9594, assigned to doubly-charged iRT peptide LGGNEQVTR by MS-GF+, SpecProb 1.9e-12). For chromatographic views call readChromatogram(rawfile, type = 'tic') to iterate over all MS1-level scans, or readChromatogram(rawfile, mass = <mz>, tol = 10, type = 'xic', filter = 'ms') for 10 ppm XICs restricted to MS1 scans; a scan filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' can be passed for fragment-ion XICs, with the C# vendor API validating and applying the filter so R needs no a priori scan-matching knowledge. Validate the resulting S3 objects with the package's validation functions before interpretation.

## Related tools

- **rawR/rawrr** (R wrapper that calls rawrr.exe via base::system2 to read spectra and chromatograms from Thermo .raw files into rawrrSpectrum and rawrrChromatogram S3 objects) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET assemblies (C#) whose API performs scan filtering and reads header, chromatographic, and spectral data from the RAW file) — https://github.com/thermofisherlsms/RawFileReader
- **rawDiag** (Proof-of-principle R package for wrapping vendor APIs to access raw files, focused on metadata analysis and LC-MS method optimization visualization) — https://github.com/fgcz/rawDiag
- **MsBackendRawFileReader** (Alternative MsBackend for Bioconductor Spectra providing direct access to spectral data in Thermo .raw files through rawrr accessor functions) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
EH4547 <- normalizePath(ExperimentHub::ExperimentHub()[["EH4547"]]); S <- rawrr::readSpectrum(EH4547, scan = 9594); plot(S[[1]]); C <- rawrr::readChromatogram(EH4547, type = "tic")
```

## Evaluation signals

- Parsed rawrrSpectrum/rawrrChromatogram objects pass the package's validation functions
- The computed TIC spans all MS1-level scans of the run (e.g. the 35 min autoQC01 run with 2.1881 x 10^4 scans on disk)
- The parsed spectrum contains numeric m/z–intensity vectors consistent with centroided, lock-mass-corrected content
- For scan 9594, all y-ion fragment signals are several ten or even hundred fold above the local noise estimate
- The TIC trace is generated by iterating over all MS1-level scans, not arbitrary scan subsets

## Limitations

- Requires rawrr.exe and the RawFileReader .NET assemblies with an appropriate runtime (.NET 8.0 on Windows, Mono on Linux/macOS); the assembly must be published per-platform with 'dotnet publish rawrr-dotnet.csproj --os <os> -a x64'
- rawR does not yet support parallel computation or exchangeable backends for data access (planned future work)
- At publication rawR was not yet submitted to Bioconductor and alignment with the R for Mass Spectrometry initiative was planned, not done
- Scan filtering is delegated to the vendor API; the R layer relies on the vendor API checking filter validity and applying the filter

## Evidence

- [results] Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet.: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [results] C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms"): "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [methods] dotnet publish rawrr-dotnet.csproj --os osx -a x64 --output /Users/cp/Library/Caches/org.R-project.R/R/rawrr/rawrrassembly/osx-x64: "dotnet publish rawrr-dotnet.csproj --os osx -a x64 --output /Users/cp/Library/Caches/org.R-project.R/R/rawrr/rawrrassembly/osx-x64"
- [readme] The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package or serves as MsRawFileReaderBackend for the Bioconductor Spectra package."
- [methods] All spectra were written to disk after applying centroiding (c) and lock mass correction: "All spectra were written to disk after applying centroiding (c) and lock mass correction"
