---
name: vendor-api-wrapping-in-r
description: Use when analysis must start from vendor-proprietary Thermo Fisher .raw
  files (e.g. 20181113_010_autoQC01.raw) in R and no mzML conversion or external preprocessor
  should be required.
when_to_use_negative:
- Input data is already converted to an open exchange format such as mzML — use general
  R MS tooling instead of the vendor wrapper
- The raw file is not from a Thermo Fisher Scientific instrument (RawFileReader only
  reads Thermo RAW files)
- A fully exchangeable Spectra backend with parallel computation is required — the
  article notes these are planned next steps, not present
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package that wraps rawrr.exe (RawFileReader) to expose readIndex, readSpectrum,
    and readChromatogram over Thermo .raw files
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET/C# assembly that actually parses header, chromatographic,
    and spectral data from Thermo RAW files on Windows, Linux, and MacOS
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Spectra MsBackend that plugs rawrr accessors into the Bioconductor Spectra
    framework as an on-disk backend
  repo: https://github.com/cpanse/MsBackendRawFileReader
- name: rawR
  role: Predecessor/prototype package demonstrating complete, OS-independent access
    to all spectral data logged in Thermo Fisher raw files
  repo: https://github.com/fgcz/rawR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:279cb3583ac3d98c707d53131c2c88b13e2b57a06d48f62533707fb0b381746d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# vendor-api-wrapping-in-r

## Summary

Directly reads proprietary Thermo Fisher Scientific Orbitraw/raw data in R by wrapping the vendor's RawFileReader .NET assembly (rawrr.exe) behind an R interface, avoiding lossy conversion to mzML via ProteoWizard.

## When to use

Use when analysis must start from vendor-proprietary Thermo Fisher .raw files (e.g. 20181113_010_autoQC01.raw) in R and no mzML conversion or external preprocessor should be required.

## When NOT to use

- Input data is already converted to an open exchange format such as mzML — use general R MS tooling instead of the vendor wrapper
- The raw file is not from a Thermo Fisher Scientific instrument (RawFileReader only reads Thermo RAW files)
- A fully exchangeable Spectra backend with parallel computation is required — the article notes these are planned next steps, not present

## Inputs

- Thermo Fisher Scientific proprietary .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547 / Bioconductor tartare)
- installed rawrr.exe runtime (via rawrr::installRawrrExe())

## Outputs

- scan index data frame from rawrr::readIndex (scan numbers, scan types, filter strings)
- rawrrSpectrum objects containing mZ, intensity, and header metadata per scan
- derived tables such as scan_deltas.csv (scan number, delta between consecutive targeted MS2 scans)

## How to apply

Install the runtime with rawrr::installRawrrExe(), which deploys the rawrr.exe wrapper compiled from the Thermo Fisher RawFileReader C# assembly (cross-compiled via 'dotnet publish rawrr-dotnet.csproj --os osx -a x64 --output' for osx/win/linux x64). Call rawrr::readIndex to retrieve scan numbers, scan types, and filter strings for the whole file (e.g. 2.1881 x 10^4 scans in the 35 min autoQC01 run), and rawrr::readSpectrum/rawrrSpectrum for individual scans such as scan 9594 (LGGNEQVTR/2, precursor 487.2567@hcd27.00). Data exchange between the R layer and the C# layer happens via file I/O, so keep the raw file path accessible and writable temp dirs. Filter targeted MS2 scans using filter strings like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and compute diffs between consecutive scan numbers to verify acquisition cycles (e.g. a constant 22-scan PRM delta). For integration with Bioconductor Spectra, attach the data through the MsBackendRawFileReader backend rather than re-implementing accessors.

## Related tools

- **rawrr** (R package that wraps rawrr.exe (RawFileReader) to expose readIndex, readSpectrum, and readChromatogram over Thermo .raw files) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET/C# assembly that actually parses header, chromatographic, and spectral data from Thermo RAW files on Windows, Linux, and MacOS) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Spectra MsBackend that plugs rawrr accessors into the Bioconductor Spectra framework as an on-disk backend) — https://github.com/cpanse/MsBackendRawFileReader
- **rawR** (Predecessor/prototype package demonstrating complete, OS-independent access to all spectral data logged in Thermo Fisher raw files) — https://github.com/fgcz/rawR

## Examples

```
idx <- rawrr::readIndex(rawfile = "20181113_010_autoQC01.raw")
```

## Evaluation signals

- readIndex returns a complete scan index matching the file's scan count (2.1881 x 10^4 scans for the 35 min autoQC01 run)
- Targeted MS2 scans matched to 'FTMS + c NSI Full ms2 487.2567@hcd27.00' show a constant diff of 22 between consecutive scan numbers
- rawrrSpectrum objects have correct-length mZ and intensity vectors (on Windows the decimal symbol must be '.' or lengths break)
- MS-GF+ identity check: scan 9594 assigned to doubly-charged LGGNEQVTR with Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4
- Spectra integration works by constructing the backend from MsBackendRawFileReader without file conversion

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol has to be configured as a '.' or mZ and intensity vectors will have incorrect lengths
- Exchangeable Spectra backends (MsBackendRawFileReader) were still in development (WIP) and parallel computation support is a planned next step, not present
- Data exchange between R and the C# layer uses file I/O, adding serialization overhead per call

## Evidence
