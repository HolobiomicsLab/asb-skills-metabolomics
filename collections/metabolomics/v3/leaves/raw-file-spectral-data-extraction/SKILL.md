---
name: raw-file-spectral-data-extraction
description: Use when raw Thermo Fisher Orbitrap .raw files must be read directly
  in R for spectra, chromatograms, or XICs without prior conversion to mzML via ProteoWizard.
when_to_use_negative:
- Data is already converted to exchange formats like mzML via ProteoWizard/MSconvert
  — use standard R/Bioconductor readers instead of vendor API access.
- Only high-level statistical postprocessing of already-aggregated outputs (e.g. MSstats
  on MaxQuant or Skyline results) is needed — no raw data reading is involved.
- Exchangeable Spectra backends are required via the R for Mass Spectrometry initiative
  — MsBackendRawFileReader is still WIP, so rawrr only serves as a backend in limited
  form.
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Bioconductor R package providing the top layer; reader functions invoke rawrr.exe
    via system2 and parse results into rawRspectrum/rawRchromatogram objects
  repo: https://github.com/fgcz/rawR
- name: Thermo Fisher RawFileReader
  role: Vendor-provided .NET/C# assemblies that the C# layer links to read header,
    chromatographic, and spectral data from RAW files on Windows, Linux, and macOS
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative MsBackend for Spectra built on rawrr accessor functions, enabling
    on-disk access to spectral data in Thermo Fisher .raw files
  repo: https://github.com/fgcz/MsBackendRawFileReader
- name: rawDiag
  role: Related R package demonstrating the proof-of-principle of wrapping vendor
    APIs for LC-MS method optimization
  repo: https://github.com/fgcz/rawDiag
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:501db25ee985710f9b34e519c5ceb7dbfa0b2b9baec6493e78d50ab0cac62e89
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
  - Thermo Fisher RawFileReader
  - rawDiag
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# raw-file-spectral-data-extraction

## Summary

Extracts spectral (m/z, intensity) and chromatographic data directly from proprietary Thermo Fisher Scientific raw files within R by delegating queries to the vendor RawFileReader API through a compiled C# wrapper (rawrr.exe) invoked via system2.

## When to use

Use when raw Thermo Fisher Orbitrap .raw files must be read directly in R for spectra, chromatograms, or XICs without prior conversion to mzML via ProteoWizard.

## When NOT to use

- Data is already converted to exchange formats like mzML via ProteoWizard/MSconvert — use standard R/Bioconductor readers instead of vendor API access.
- Only high-level statistical postprocessing of already-aggregated outputs (e.g. MSstats on MaxQuant or Skyline results) is needed — no raw data reading is involved.
- Exchangeable Spectra backends are required via the R for Mass Spectrometry initiative — MsBackendRawFileReader is still WIP, so rawrr only serves as a backend in limited form.

## Inputs

- Proprietary Thermo Fisher Scientific Orbitrap .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab, obtainable via ExperimentHub entry EH4547 / Bioconductor package tartare or MassIVE dataset MSV000086542)
- Query arguments: scan numbers, mass ranges, and trace filters (e.g. 'ms', 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')

## Outputs

- S3 object rawRspectrum containing m/z and intensity vectors plus header keys for a queried scan
- S3 object rawRchromatogram (and set collections thereof) for total ion chromatograms or XICs
- Exported parsed content, e.g. spectrum_chromatogram_data.csv with length-matched m/z and intensity vectors

## How to apply

Set up a two-layer architecture: an R top layer and a C#/.NET bottom layer that links the ThermoFisher.CommonCore assemblies from the vendor RawFileReader repository. Compile rawrr.exe via 'dotnet publish rawrr-dotnet.csproj -c Release' cross-compiling for osx-x64, win-x64, and linux-x64 (rawrr::installRawrrExe() automates runtime installation; Mono is needed on Linux/macOS). R reader functions call the executable via base::system2, passing the raw file path plus query arguments such as scan numbers and mass ranges. The C# layer writes extracted data (m/z and intensity tuples, header keys) to a temporary location on disk, which R reads back and parses into S3 objects named rawRspectrum and rawRchromatogram. Chromatographic traces such as XICs are not computed in R: they are calculated by a C# method that calls the vendor API, which handles filter validity checking and filter application (e.g. filter = 'ms' for MS1-only XICs, 10 ppm tolerance around the target mass, or 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]').

## Related tools

- **rawrr** (Bioconductor R package providing the top layer; reader functions invoke rawrr.exe via system2 and parse results into rawRspectrum/rawRchromatogram objects) — https://github.com/fgcz/rawR
- **Thermo Fisher RawFileReader** (Vendor-provided .NET/C# assemblies that the C# layer links to read header, chromatographic, and spectral data from RAW files on Windows, Linux, and macOS) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative MsBackend for Spectra built on rawrr accessor functions, enabling on-disk access to spectral data in Thermo Fisher .raw files) — https://github.com/fgcz/MsBackendRawFileReader
- **rawDiag** (Related R package demonstrating the proof-of-principle of wrapping vendor APIs for LC-MS method optimization) — https://github.com/fgcz/rawDiag

## Examples

```
library(ExperimentHub); eh <- ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); rs <- rawrr::readSpectrum(rawfile, 9594); xic <- rawrr::readChromatogram(rawfile, c(560.26, 10), type = "xic", filter = "ms")
```

## Evaluation signals

- Reader runs end-to-end without error on the example file 20181113_010_autoQC01.raw (MD5 a1f5df9627cf9e0d51ec1906776957ab).
- Exported table contains non-empty, length-matched m/z and intensity vectors for the extracted scans (e.g. scan 9594).
- Total scan count for the 35 min run matches the paper's reported 2.1881 x 10^4 scans.
- Scan 9594 spectrum corresponds to the doubly-charged iRT peptide LGGNEQVTR (MS-GF+ Score 144, SpecProb 1.9e-12), with y-ion signals tens to hundreds of folds above noise.
- Class instances pass the package's validation functions (validate_ functions on rawRspectrum/rawRchromatogram).

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version.
- On Windows, the decimal symbol must be configured as '.' for m/z and intensity vectors to have correct lengths.
- Parallel computation support is a planned next step, not present in the current implementation.
- Full exchangeable Spectra backend integration (MsBackendRawFileReader) is planned but not yet complete.

## Evidence

- [readme] RawFileReader assemblies are cross-platform C# readers of Thermo RAW files: [evidence span withheld — anchor status: external_doc]
