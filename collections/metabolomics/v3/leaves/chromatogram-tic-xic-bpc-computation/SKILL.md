---
name: chromatogram-tic-xic-bpc-computation
description: Use when the input is a proprietary Thermo Fisher Scientific .raw file
  and the analysis goal is TIC/XIC/BPC traces from within R, especially to avoid ProteoWizard/mzML
  conversion steps.
when_to_use_negative:
- Input data has already been converted to an exchange format such as mzML — use conventional
  R readers (e.g., mzR) instead of the vendor-API path
- Only scan-level spectra or header metadata are needed (use readSpectrum/rawrrSpectrum
  readers, not readChromatogram)
- Input files are not Thermo Fisher Scientific .raw files — the RawFileReader API
  only reads this vendor's proprietary format
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package whose reader functions call rawrr.exe via base::system2 and parse
    returned chromatogram data into S3 objects
  repo: https://github.com/fgcz/rawrr
- name: Thermo Fisher RawFileReader
  role: Vendor .NET assemblies (C#) that the C# wrapper method calls to compute chromatographic
    traces and apply filter validity checks
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: rawR
  role: Predecessor package demonstrating the same wrapped-vendor-API access pattern
    for spectral/chromatographic data
  repo: https://github.com/fgcz/rawR
- name: tartare
  role: Bioconductor data package supplying example raw files through ExperimentHub
    entry EH4547
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:5963692348d110bcd0d53042a771966341cb9f5678e6cc0d010411e98cb0edea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - .NET
  - C#
  - Mono
  - R
  - Thermo Fisher RawFileReader
  - rawR
  - rawrr
  - tartare
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# chromatogram-tic-xic-bpc-computation

## Summary

Computes total ion, extracted-ion, and base-peak chromatograms (TIC/XIC/BPC) directly from proprietary Thermo Fisher .raw files inside R by delegating the trace calculation to a compiled C# method (rawrr.exe) that calls the vendor RawFileReader API. This avoids prior mzML conversion and lets the vendor API handle filter validation and application.

## When to use

Use when the input is a proprietary Thermo Fisher Scientific .raw file and the analysis goal is TIC/XIC/BPC traces from within R, especially to avoid ProteoWizard/mzML conversion steps.

## When NOT to use

- Input data has already been converted to an exchange format such as mzML — use conventional R readers (e.g., mzR) instead of the vendor-API path
- Only scan-level spectra or header metadata are needed (use readSpectrum/rawrrSpectrum readers, not readChromatogram)
- Input files are not Thermo Fisher Scientific .raw files — the RawFileReader API only reads this vendor's proprietary format

## Inputs

- Proprietary Thermo Fisher Scientific .raw file (e.g., 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Query arguments: chromatogram type (tic/xic/bpc), target m/z values, mass tolerance (10 ppm for XIC), filter string (e.g., 'ms' for MS1-level scans or an instrument filter like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')

## Outputs

- rawRchromatogram / rawRchromatogramSet S3 objects (times and intensity vectors per trace)
- TIC built by iterating over all MS1-level scans of the run (35 min run yielded 2.1881 x 10^4 scans)

## How to apply

Obtain the raw file path (e.g., 20181113_010_autoQC01.raw via ExperimentHub entry EH4547 from the tartare Bioconductor package or MassIVE MSV000086542) and ensure rawrr.exe is compiled (dotnet publish for osx-x64/win-x64/linux-x64) with the ThermoFisher.CommonCore assemblies from thermofisherlsms/RawFileReader. Call the R layer (e.g., readChromatogram) which invokes the executable via base::system2; chromatographic traces such as XICs are NOT computed within R — they are calculated by the C# method that calls the vendor API, and the API takes care of checking filter validity and applying the filter. For XICs, request a mass tolerance of 10 ppm around the target mass and restrict to MS1-level scans by setting filter = 'ms'; for fragment-ion traces pass the instrument filter string (e.g., 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'). The C# layer writes the extracted chromatogram data to a temporary file, which R reads back and parses into a rawRchromatogram (or collection) S3 object. Rationale: calling the vendor API directly preserves instrument-native filtering semantics and keeps the whole pipeline (raw reading + statistics) modular in R, in line with the Bioconductor ecosystem.

## Related tools

- **rawrr** (R package whose reader functions call rawrr.exe via base::system2 and parse returned chromatogram data into S3 objects) — https://github.com/fgcz/rawrr
- **Thermo Fisher RawFileReader** (Vendor .NET assemblies (C#) that the C# wrapper method calls to compute chromatographic traces and apply filter validity checks) — https://github.com/thermofisherlsms/RawFileReader
- **rawR** (Predecessor package demonstrating the same wrapped-vendor-API access pattern for spectral/chromatographic data) — https://github.com/fgcz/rawR
- **tartare** (Bioconductor data package supplying example raw files through ExperimentHub entry EH4547)

## Examples

```
eh <- ExperimentHub::ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); chr <- rawrr::readChromatogram(rawfile, type = "xic", filter = "ms")
```

## Evaluation signals

- For XICs, confirm the call restricted input to MS1-level scans (filter = 'ms') and used a 10 ppm tolerance around the target mass
- The chromatogram object returns non-empty, length-matched time/intensity vectors; on Windows verify the decimal symbol is configured as '.' since wrong locale settings can corrupt vector lengths
- An invalid filter string should be rejected by the vendor API layer (the API takes care of checking filter validity), not silently return empty traces
- A TIC spanning the full run should cover 2.1881 x 10^4 scans for the 35 min autoQC01 example file
- The end-to-end reader call completes without error and the returned object passes the package's validation functions

## Limitations

- On Windows, the decimal symbol must be configured as '.' or m/z and intensity vectors can come back with incorrect lengths
- On Linux/macOS the Mono runtime must be available to execute the rawrr.exe wrapper
- Parallel computation and exchangeable Spectra backends (MsBackendRawFileReader alignment) were planned next steps, not available at publication time

## Evidence

- [methods] XIC filtering delegated to C#/vendor API: [evidence span withheld — anchor status: not_found]
- [results] XIC 10 ppm tolerance, MS1-level scans only: "Each XIC was calculated using a tolerance of 10 ppm around the target mass and using only MS1-level scans."
- [results] Instrument filter string for fragment-ion traces: "filter = "FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]""
- [discussion] Windows decimal symbol caveat: "On Windows, the decimal symbol has to be configured as a '.'!"
- [methods] File I/O return path from C# to R: "In order to return extracted data back to the `R` layer we use file I/O"
- [readme] rawrr wraps RawFileReader .NET functionality: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
