---
name: s3-object-parsing-from-file-io
description: Use when an R workflow shells out to rawrr.exe (or another vendor-API
  wrapper) that writes spectral or chromatographic results to a temporary location
  on disk, and those files must be read back and structured into typed S3 objects
  for downstream analysis or plotting.
when_to_use_negative:
- The input data is already in an exchange format such as mzML — use standard R/Bioconductor
  mzML readers instead of the vendor-API wrapper
- Only scan header/metadata is needed and no spectra or chromatograms will be parsed
  (readFileHeader() suffices)
- No Thermo Fisher .raw file and no rawrr.exe/.NET (or Mono) runtime are available
  — the two-layer mechanism cannot execute
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Provides readSpectrum()/readChromatogram() wrappers that call rawrr.exe via
    system2 and parse the temporary output into rawrrSpectrum and rawrrChromatogram
    S3 objects
  repo: https://github.com/fgcz/rawrr
- name: Thermo Fisher RawFileReader
  role: Vendor .NET assemblies (C#) that the rawrr.exe assembly uses to read scan
    filtering and spectral data from .raw files before results are written to temporary
    files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Consumes the rawrr accessors as an on-disk MsBackend for the Bioconductor
    Spectra package
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:e92fd0990635cd2143368c244cf84b8822347beab6d52d2ebe74194fb7d8060c
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
yes_rate: 0.8
n_claims: 15
---

# s3-object-parsing-from-file-io

## Summary

Parse the temporary-file output written by rawrr.exe (invoked via base::system2) back into R memory as validated S3 objects — rawrrSpectrum/rawrrSpectrumSet for centroided scans and rawrrChromatogram for TIC/XIC traces — completing the two-layer R/C# data-access bridge to Thermo Fisher .raw files.

## When to use

Use when an R workflow shells out to rawrr.exe (or another vendor-API wrapper) that writes spectral or chromatographic results to a temporary location on disk, and those files must be read back and structured into typed S3 objects for downstream analysis or plotting.

## When NOT to use

- The input data is already in an exchange format such as mzML — use standard R/Bioconductor mzML readers instead of the vendor-API wrapper
- Only scan header/metadata is needed and no spectra or chromatograms will be parsed (readFileHeader() suffices)
- No Thermo Fisher .raw file and no rawrr.exe/.NET (or Mono) runtime are available — the two-layer mechanism cannot execute

## Inputs

- Thermo Fisher Scientific proprietary .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Temporary output files written to disk by rawrr.exe after a system2 call
- Filter strings (e.g. 'ms' or 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]') and m/z target lists with ppm tolerance for XICs

## Outputs

- rawrrSpectrum or rawrrSpectrumSet S3 objects with numeric m/z and intensity vectors for centroided scans
- rawrrChromatogram S3 objects (TIC over all MS1-level scans, or XICs at e.g. 10 ppm tolerance)
- Validated object instances ready for print/plot generics and downstream Bioconductor (Spectra/MsBackendRawFileReader) use

## How to apply

From R, call base::system2 on the rawrr.exe assembly (built via 'dotnet publish rawrr-dotnet.csproj --os <os> -a x64' and shipped with the ThermoFisher.CommonCore RawFileReader .NET assemblies, run under .NET 8.0 on Windows or Mono on Linux/macOS) against a .raw file such as 20181113_010_autoQC01.raw, requesting spectrum or chromatogram data. The C# layer delegates scan filtering to the vendor API (which checks filter validity and applies the filter string, e.g. 'ms' for MS1-only or a full 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' rule), so no a priori knowledge of matching scans is needed at the R level. Read the temporary output files back into memory and parse them into S3 objects: readSpectrum() returns a rawrrSpectrum or rawrrSpectrumSet holding numeric m/z–intensity vectors for selected centroided, lock-mass-corrected scans, while readChromatogram(type = 'tic') iterates over all MS1-level scans to build a rawrrChromatogram (or type = 'xic' with a mass list and ppm tolerance). Run the package's validation functions on each constructed instance before applying print/plot generics.

## Related tools

- **rawrr** (Provides readSpectrum()/readChromatogram() wrappers that call rawrr.exe via system2 and parse the temporary output into rawrrSpectrum and rawrrChromatogram S3 objects) — https://github.com/fgcz/rawrr
- **Thermo Fisher RawFileReader** (Vendor .NET assemblies (C#) that the rawrr.exe assembly uses to read scan filtering and spectral data from .raw files before results are written to temporary files) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Consumes the rawrr accessors as an on-disk MsBackend for the Bioconductor Spectra package) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")
```

## Evaluation signals

- Parsed objects pass the rawrr package validation functions without errors
- The rawrrChromatogram TIC spans the MS1-level scans of the run (e.g. 2.1881 × 10^4 scans for the 35 min autoQC01 run)
- Parsed rawrrSpectrum contains numeric m/z–intensity vectors consistent with the centroided, lock-mass-corrected content of the sample file
- Known scans reconcile with external assignments (e.g. scan 9594 assigned to doubly-charged iRT peptide LGGNEQVTR, MS-GF+ Score 144, SpecProb 1.9e-12)
- Input file integrity confirmed via MD5 a1f5df9627cf9e0d51ec1906776957ab before parsing

## Limitations

- Requires rawrr.exe plus ThermoFisher.CommonCore RawFileReader assemblies and a .NET 8.0 (Windows) or Mono (Linux/macOS) runtime; the wrapper writes intermediate results to disk, adding file-I/O overhead
- At the described release, rawR lacks exchangeable backend support and parallel computation, and had not yet been submitted to Bioconductor
- Scan filtering correctness depends entirely on the vendor API validating and applying the filter string; an invalid filter yields no matching scans rather than an R-level error

## Evidence

- [methods] From R, call base::system2 on rawrr.exe to request spectrum and chromatogram data from the raw file, with the extracted information written by the wrapper to a temporary location on the harddrive.: [evidence span withheld — anchor status: not_found]
- [methods] Read the temporary output back into memory and parse it into S3 objects: a rawrrSpectrum(-Set) for selected centroided scans and a rawrrChromatogram for the total ion chromatogram generated by iterating over all MS1-level scans.: [evidence span withheld — anchor status: not_found]
- [results] Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet.: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [other] the vendor API takes care of the scan filtering process (checks filter validity and applies the filter), so R-level code needs no a priori knowledge of which scans match the filter rule.: [evidence span withheld — anchor status: not_found]
- [readme] rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
