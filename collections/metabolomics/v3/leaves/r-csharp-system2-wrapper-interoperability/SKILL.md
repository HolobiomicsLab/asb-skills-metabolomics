---
name: r-csharp-system2-wrapper-interoperability
description: Use when an R package must read proprietary vendor binary formats (e.g.
  Thermo Fisher .raw Orbitrap files) for which no native R parser exists, by delegating
  queries to a vendor-provided .NET/C# API.
when_to_use_negative:
- The input is already an open exchange format such as mzML that R libraries can parse
  directly without a vendor API
- The data are protein/peptide-level aggregates (e.g. MaxQuant or Skyline outputs
  processed by MSstats) rather than raw spectral or chromatographic data
- Parallel or exchangeable Spectra backend access is required — these are planned
  next steps, not yet implemented
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: 'R package implementing the top layer: calls the compiled rawrr.exe via system2
    and parses returned files into rawRspectrum / rawRchromatogram objects'
  repo: https://github.com/fgcz/rawrr
- name: Thermo Fisher RawFileReader
  role: Vendor .NET assemblies (C#) used by the compiled wrapper to read header, chromatographic,
    and spectral data from .raw files on Windows, Linux, and macOS
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: Mono
  role: Runtime required to execute the rawrr.exe wrapper on Linux and macOS
- name: MSFileReader
  role: Older vendor product replaced by RawFileReader; not used by this skill
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:0708aa8d0f7c8905cf99d149af1454d517f9a344f0c53ab00109ea78e0067bcd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - .NET
  - C#
  - MSFileReader
  - Mono
  - R
  - Thermo Fisher RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# r-csharp-system2-wrapper-interoperability

## Summary

A two-layer R/C# interoperability pattern in which an R package delegates raw Thermo Fisher Orbitrap file queries to a compiled .NET executable (rawrr.exe, built on the vendor RawFileReader assemblies) via base::system2, exchanging data through temporary on-disk files that R parses back into S3 objects such as rawRspectrum and rawRchromatogram.

## When to use

Use when an R package must read proprietary vendor binary formats (e.g. Thermo Fisher .raw Orbitrap files) for which no native R parser exists, by delegating queries to a vendor-provided .NET/C# API.

## When NOT to use

- The input is already an open exchange format such as mzML that R libraries can parse directly without a vendor API
- The data are protein/peptide-level aggregates (e.g. MaxQuant or Skyline outputs processed by MSstats) rather than raw spectral or chromatographic data
- Parallel or exchangeable Spectra backend access is required — these are planned next steps, not yet implemented

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Query arguments: scan numbers (e.g. scan 9594), mass ranges, filter strings, XIC tolerances

## Outputs

- S3 objects rawRspectrum and rawRchromatogram (with set collections rawSpectrum / rawChromatogram)
- m/z and intensity vectors plus spectrum header keys extracted from the raw file
- Exportable CSV table of spectrum and chromatogram content

## How to apply

Compile the C# wrapper (rawrr.exe) from source with `dotnet publish rawrr-dotnet.csproj` cross-compiling for osx-x64, win-x64, and linux-x64, linking the ThermoFisher.CommonCore assemblies from the RawFileReader repository; on Linux and macOS the mono runtime must be available. From the R layer, call the compiled executable with base::system2, passing the .raw file path and query arguments (scan numbers, mass ranges, filter strings). The C# layer writes extracted spectral and chromatographic data (m/z and intensity tuples plus header keys) to a temporary file on the harddrive, which R reads back and parses into S3 objects named rawRspectrum and rawRchromatogram, plus their set collections. On Windows, the decimal symbol must be configured as '.' or the mZ and intensity vectors will not have correct lengths. Cross-compile for all three OS targets to keep the package OS independent, and validate returned class instances with the package's validation functions.

## Related tools

- **rawrr** (R package implementing the top layer: calls the compiled rawrr.exe via system2 and parses returned files into rawRspectrum / rawRchromatogram objects) — https://github.com/fgcz/rawrr
- **Thermo Fisher RawFileReader** (Vendor .NET assemblies (C#) used by the compiled wrapper to read header, chromatographic, and spectral data from .raw files on Windows, Linux, and macOS) — https://github.com/thermofisherlsms/RawFileReader
- **Mono** (Runtime required to execute the rawrr.exe wrapper on Linux and macOS)
- **MSFileReader** (Older vendor product replaced by RawFileReader; not used by this skill)

## Examples

```
Rscript -e 'rawrr::installRawrrExe(); r <- rawrr::readSpectrum(system.file("extdata","20181113_010_autoQC01.raw",package="tartare"), 9594)'
```

## Evaluation signals

- rawrr.exe builds via `dotnet publish` for osx-x64, win-x64, and linux-x64 and executes on all three OS platforms (mono runtime on Linux/macOS)
- Reader runs end-to-end on the test file 20181113_010_autoQC01.raw whose MD5 checksum is a1f5df9627cf9e0d51ec1906776957ab
- Parsed objects are validated rawRspectrum / rawRchromatogram class instances with non-empty, length-matched m/z and intensity vectors
- The run contains 2.1881 x 10^4 scans; a full iteration over MS1-level scans yields a total ion chromatogram, and scan 9594 extracts a spectrum for peptide LGGNEQVTR/2
- On Windows, decimal symbol configured as '.' so mZ and intensity vectors have correct lengths

## Limitations

- On Windows the decimal symbol must be set to '.', otherwise the mZ and intensity vectors will not have correct lengths
- The mono runtime is required to execute the wrapper on Linux and macOS
- The rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- Parallel computation support and an exchangeable Spectra backend (MsBackendRawFileReader is still WIP) are planned but not yet available
- MSFileReader, the older vendor product, is replaced by RawFileReader

## Evidence

- [methods] Chromatographic traces such as XICs are not computed within R; instead they are directly calculated by a C# method that calls the vendor API: [evidence span withheld — anchor status: not_found]
