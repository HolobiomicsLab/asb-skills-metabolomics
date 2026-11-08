---
name: temporary-file-io-data-exchange-parsing
description: Use when an R process must ingest results produced by an external compiled
  executable (e.g. a .NET wrapper around the Thermo RawFileReader API) via temporary
  files, then parse them into typed S3 objects.
when_to_use_negative:
- The raw data is already converted to an open exchange format like mzML via ProteoWizard/MSconvert
  — a converter-based reader is the simpler path
- The input is not a proprietary Thermo Fisher .raw file (RawFileReader only reads
  Thermo RAW files)
- You need exchangeable Spectra backends or parallel data access — the article lists
  these as planned, not implemented
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package whose reader functions call rawrr.exe via base::system2 and parse
    the temporary file output into rawRspectrum/rawRchromatogram objects
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor .NET assemblies the C# layer links against to read header, chromatographic,
    and spectral data from Thermo .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Spectra backend built on rawrr's parsed output as an on-disk MsBackend
  repo: https://github.com/fgcz/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:3667f623b04cd036f276b2f31b79d119f3afdf7c40be32d087d31a472ed51ee8
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# temporary-file-io-data-exchange-parsing

## Summary

Implements the file-I/O handoff used by rawrr/rawR: the compiled C# wrapper (rawrr.exe) writes extracted spectral and chromatographic data (m/z, intensity tuples and header keys) to a temporary location on disk, and the R layer reads it back and parses it into rawRspectrum / rawRchromatogram S3 objects.

## When to use

Use when an R process must ingest results produced by an external compiled executable (e.g. a .NET wrapper around the Thermo RawFileReader API) via temporary files, then parse them into typed S3 objects.

## When NOT to use

- The raw data is already converted to an open exchange format like mzML via ProteoWizard/MSconvert — a converter-based reader is the simpler path
- The input is not a proprietary Thermo Fisher .raw file (RawFileReader only reads Thermo RAW files)
- You need exchangeable Spectra backends or parallel data access — the article lists these as planned, not implemented

## Inputs

- Proprietary Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab from ExperimentHub EH4547 / MassIVE MSV000086542)
- Query arguments: scan numbers (e.g. 9594), mass ranges, filter strings
- Compiled rawrr.exe (dotnet publish for osx-x64, win-x64, linux-x64)

## Outputs

- S3 object rawRspectrum (m/z and intensity vectors plus header keys for a scan)
- S3 object rawRchromatogram (e.g. XIC or total ion chromatogram assembled from MS1-level scans)
- Exported table such as spectrum_chromatogram_data.csv

## How to apply

Have the R top layer invoke the compiled rawrr.exe via base::system2, passing the Thermo Fisher .raw file path plus query arguments (scan numbers, mass ranges, filter strings such as 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' or 'ms' for MS1-only XICs at 10 ppm tolerance). Rather than streaming results over a pipe, the C# layer writes the extracted m/z and intensity tuples and header keys to a temporary location on the harddrive, and the R layer reads those files back and parses them into S3 objects named rawRspectrum and rawRchromatogram plus their set collections. Validate the returned instances with the package's validation functions before downstream use; chromatographic traces such as XICs are computed by the C# method calling the vendor API, not in R. Ensure runtime prerequisites (rawrr::installRawrrExe(), mono on Linux/macOS) are met before any query.

## Related tools

- **rawrr** (R package whose reader functions call rawrr.exe via base::system2 and parse the temporary file output into rawRspectrum/rawRchromatogram objects) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor .NET assemblies the C# layer links against to read header, chromatographic, and spectral data from Thermo .raw files) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Spectra backend built on rawrr's parsed output as an on-disk MsBackend) — https://github.com/fgcz/MsBackendRawFileReader

## Examples

```
library(ExperimentHub); eh <- ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); rawrr::installRawrrExe()
```

## Evaluation signals

- The reader runs end-to-end without error on the EH4547 example file and the exported table contains non-empty, length-matched m/z and intensity vectors
- The parsed spectrum for scan 9594 matches the article's expectation (LGGNEQVTR/2, y-ion signals several ten to hundred folds above the noise estimate) in a 35 min run of 2.1881 × 10^4 scans
- Returned objects pass the package's validation functions as rawRspectrum / rawRchromatogram class instances
- On Windows, m/z and intensity vectors have correct lengths only if the decimal symbol is configured as '.' — mismatched lengths indicate a locale failure
- XIC filter queries return traces computed only from MS1-level scans (filter = 'ms') within the specified 10 ppm tolerance

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows the decimal symbol must be configured as '.' or m/z and intensity vectors will not have correct lengths
- On Linux/macOS the mono runtime must be available to execute the rawrr.exe wrapper; parallel computation support is a planned next step, not present

## Evidence
