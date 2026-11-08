---
name: dotnet-cross-compilation-and-runtime-bundling
description: Use when an R (or other scripting-language) package must execute a compiled
  .NET/C# vendor library cross-platform, and the wrapper binary must be pre-built
  and bundled rather than compiled on the user machine.
when_to_use_negative:
- When the user environment is Windows-only with Visual Studio available and no bundled
  binary is desired - compiling natively is simpler.
- When the downstream consumer already uses the rawrr package and never needs to rebuild
  the wrapper from source.
- When the vendor library is pure R or Python and a .NET toolchain is unnecessary.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: dotnet
  role: Cross-compiles the C# wrapper via 'dotnet publish rawrr-dotnet.csproj -c Release'
    for osx-x64, win-x64, and linux-x64.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: mono
  role: Runtime required to execute the compiled wrapper on Linux and macOS.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: RawFileReader
  role: Vendor .NET assemblies (C#) that the compiled wrapper links against to read
    Thermo Fisher raw files.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: rawrr
  role: R package that bundles the published binary and installs it at runtime via
    installRawrrExe().
  repo: https://github.com/fgcz/rawrr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:e8bd312830b00f3a13c174d64deb72aabdbcb1a1e89192730bbbf93338990a96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/thermofisherlsms/RawFileReader
  tools:
  - .NET
  - C#
  - Mono
  - R
  - RawFileReader
  - Thermo Fisher RawFileReader
  - dotnet
  - mono
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# dotnet-cross-compilation-and-runtime-bundling

## Summary

Compiles a C#/.NET wrapper (rawrr.exe) from source with the Thermo Fisher CommonCore assemblies for osx-x64, win-x64, and linux-x64, and ships it inside an R package so callers can invoke vendor APIs without a local .NET toolchain.

## When to use

Use when an R (or other scripting-language) package must execute a compiled .NET/C# vendor library cross-platform, and the wrapper binary must be pre-built and bundled rather than compiled on the user machine.

## When NOT to use

- When the user environment is Windows-only with Visual Studio available and no bundled binary is desired - compiling natively is simpler.
- When the downstream consumer already uses the rawrr package and never needs to rebuild the wrapper from source.
- When the vendor library is pure R or Python and a .NET toolchain is unnecessary.

## Inputs

- C# wrapper source (rawrr-dotnet.csproj / RawFileReader example code)
- ThermoFisher.CommonCore assemblies (NuGet/libs folder of RawFileReader)
- Target platform specification (osx-x64, win-x64, linux-x64)

## Outputs

- rawrr.exe binaries for osx-x64, win-x64, and linux-x64
- Standalone executable bundled into the R package for runtime installation

## How to apply

Compile the C# wrapper (rawrr.exe) from source using 'dotnet publish rawrr-dotnet.csproj -c Release', cross-compiling for osx-x64, win-x64, and linux-x64, and link the ThermoFisher.CommonCore assemblies obtained from github.com/thermofisherlsms/RawFileReader. On Linux and macOS the mono runtime must be present to execute the compiled wrapper; within the R package the runtime can be installed on demand via rawrr::installRawrrExe(). Build reproducibility is validated by a Dockerfile that performs R CMD build and R CMD check with the published binary in place. The resulting executable is invoked from R via base::system2, passing the raw file path and query arguments (scan numbers, mass ranges), and it writes extracted spectral and chromatographic data to disk for R to read back.

## Related tools

- **dotnet** (Cross-compiles the C# wrapper via 'dotnet publish rawrr-dotnet.csproj -c Release' for osx-x64, win-x64, and linux-x64.) — https://github.com/thermofisherlsms/RawFileReader
- **mono** (Runtime required to execute the compiled wrapper on Linux and macOS.) — https://github.com/thermofisherlsms/RawFileReader
- **RawFileReader** (Vendor .NET assemblies (C#) that the compiled wrapper links against to read Thermo Fisher raw files.) — https://github.com/thermofisherlsms/RawFileReader
- **rawrr** (R package that bundles the published binary and installs it at runtime via installRawrrExe().) — https://github.com/fgcz/rawrr

## Examples

```
dotnet publish rawrr-dotnet.csproj -c Release --os osx -a x64 --output ./bin; Rscript -e 'rawrr::installRawrrExe()'
```

## Evaluation signals

- The wrapper compiles without error for all three targets (osx-x64, win-x64, linux-x64) using 'dotnet publish rawrr-dotnet.csproj -c Release'.
- R CMD check of the Dockerfile-based build passes with the bundled binary in place.
- rawrr::installRawrrExe() completes and the resulting rawrr.exe runs end-to-end on the target OS (mono required on Linux/macOS).
- Invoking the wrapper with a real .raw file (e.g. 20181113_010_autoQC01.raw) yields non-empty m/z and intensity vectors that R parses back into rawRspectrum / rawRchromatogram objects.

## Limitations

- The mono runtime must be available on Linux and macOS to execute the compiled wrapper.
- Compilation and bundling depend on ThermoFisher.CommonCore assemblies fetched from the vendor RawFileReader GitHub repo (or NuGet/libs folder), so upstream changes can break the build.
- On Windows, the decimal symbol must be configured as '.' for m/z and intensity vectors to have correct lengths.

## Evidence

- [methods] dotnet publish rawrr-dotnet.csproj -c Release: "dotnet publish rawrr-dotnet.csproj --os osx -a x64 --output"
- [methods] rawrr::installRawrrExe(): "rawrr::installRawrrExe()"
- [readme] RawFileReader assembly usage: [evidence span withheld — anchor status: external_doc]
