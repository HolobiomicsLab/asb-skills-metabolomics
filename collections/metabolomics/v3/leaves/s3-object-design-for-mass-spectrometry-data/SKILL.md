---
name: s3-object-design-for-mass-spectrometry-data
description: Use when implementing an R package or pipeline that reads Thermo Fisher
  Orbitrap .raw files via the vendor RawFileReader API and must parse exchanged m/z/intensity
  tuples and scan headers into validated S3 objects for downstream R analysis.
when_to_use_negative:
- Input is already an open exchange format such as mzML — use converter-based readers
  (ProteoWizard/MSconvert) or generic parsers instead of the vendor-API S3 wrapper
- You need an exchangeable Spectra backend with full Spectra-package accessors — MsBackendRawFileReader
  was only planned/not yet implemented at writing, so direct Spectra integration is
  out of scope
- Parallelized raw-data access across many raw files is required — parallel computation
  is a planned next step and not present in rawrr
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: rawrr
  role: Provides the R layer whose system2 calls to rawrr.exe return file-I/O content
    parsed into the rawrrSpectrum/rawrrChromatogram S3 objects
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor .NET assemblies (C#) compiled into rawrr.exe that read header, chromatographic,
    and spectral data from the .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Host environment defining the S3 classes, validation functions, and system2/file-I/O
    plumbing
- name: MsBackendRawFileReader
  role: Planned alternative MsBackend to Spectra built on rawrr accessors; relevant
    as the intended downstream integration of the S3 objects
  repo: https://github.com/fgcz/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:d1bdb7dad4705584f5e2eefd03f1c1769d4d6f9db109e31ed7b5a0b8ced4c234
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - .NET
  - C#
  - Mono
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - Thermo Fisher RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# s3-object-design-for-mass-spectrometry-data

## Summary

Design of R S3 object classes (rawrrSpectrum, rawrrChromatogram and their set collections) that wrap spectral and chromatographic data extracted from proprietary Thermo Fisher .raw files by the rawrr C#/.NET rawrr.exe layer, validated with dedicated package functions.

## When to use

Use when implementing an R package or pipeline that reads Thermo Fisher Orbitrap .raw files via the vendor RawFileReader API and must parse exchanged m/z/intensity tuples and scan headers into validated S3 objects for downstream R analysis.

## When NOT to use

- Input is already an open exchange format such as mzML — use converter-based readers (ProteoWizard/MSconvert) or generic parsers instead of the vendor-API S3 wrapper
- You need an exchangeable Spectra backend with full Spectra-package accessors — MsBackendRawFileReader was only planned/not yet implemented at writing, so direct Spectra integration is out of scope
- Parallelized raw-data access across many raw files is required — parallel computation is a planned next step and not present in rawrr

## Inputs

- Thermo Fisher Scientific proprietary .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab, via ExperimentHub EH4547 / tartare or MassIVE MSV000086542)
- Query arguments: scan numbers (e.g. scan 9594), mass ranges, or chromatogram filter strings (e.g. 'ms' with 10 ppm tolerance)
- Temp-file output of the C# rawrr.exe layer containing m/z, intensity tuples and scan header keys

## Outputs

- R S3 objects of class rawrrSpectrum (m/z and intensity vectors plus scan header keys)
- R S3 objects of class rawrrChromatogram (e.g. XIC or TIC traces)
- Set/collection objects of the same S3 classes for multiple scans
- Exported tabular content such as spectrum_chromatogram_data.csv

## How to apply

Follow the rawrr two-layer architecture: the R top layer calls the compiled C# rawrr.exe wrapper via base::system2, passing the raw file path and query arguments such as scan numbers or mass ranges; the C# layer calls the ThermoFisher.CommonCore RawFileReader assemblies, which also perform filter validity checks and apply filters (e.g. XICs computed with 10 ppm tolerance using only MS1-level scans, filter = 'ms'). Extracted data (m/z, intensity tuples and header keys) are written to a temporary location on the hard drive by the C# layer and read back by R via file I/O. Parse the returned content into S3 classes named rawRspectrum and rawrrChromatogram, plus their set/collection variants, and run the package's validation functions on each instance. Validate end-to-end by extracting a total ion chromatogram by iterating over all MS1-level scans and a single spectrum (e.g. scan 9594 from 20181113_010_autoQC01.raw), confirming non-empty, length-matched m/z and intensity vectors consistent with the reported 2.1881 x 10^4 scans of the 35 min run.

## Related tools

- **rawrr** (Provides the R layer whose system2 calls to rawrr.exe return file-I/O content parsed into the rawrrSpectrum/rawrrChromatogram S3 objects) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor .NET assemblies (C#) compiled into rawrr.exe that read header, chromatographic, and spectral data from the .raw files) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Host environment defining the S3 classes, validation functions, and system2/file-I/O plumbing)
- **MsBackendRawFileReader** (Planned alternative MsBackend to Spectra built on rawrr accessors; relevant as the intended downstream integration of the S3 objects) — https://github.com/fgcz/MsBackendRawFileReader

## Examples

```
rawrr::installRawrrExe(); eh <- ExperimentHub::ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(rawfile, 9594)
```

## Evaluation signals

- Reader runs end-to-end without error on the example file (EH4547 / 20181113_010_autoQC01.raw)
- Exported table (spectrum_chromatogram_data.csv) contains non-empty, length-matched m/z and intensity vectors for extracted scans
- Total scans read back are consistent with the expected 2.1881 x 10^4 scans of the 35 min autoQC run
- Class instances pass the package's validation functions (rawrrSpectrum / rawrrChromatogram validators)
- TIC extracted by iterating over all MS1-level scans matches trace computed by the C# layer with the ms-level filter

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' or the m/z and intensity vectors will not have correct lengths
- rawrr.exe must be cross-compiled per platform (osx-x64, win-x64, linux-x64) and requires the mono runtime on Linux/macOS; parallel computation support is not yet implemented

## Evidence
