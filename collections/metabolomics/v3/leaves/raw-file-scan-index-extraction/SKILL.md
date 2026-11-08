---
name: raw-file-scan-index-extraction
description: Use when the input is a Thermo Fisher Scientific Orbitrap .raw file (e.g.,
  20181113_010_autoQC01.raw) and you must enumerate, filter, or select scans (MS1-only
  or targeted MS2) before per-scan spectral extraction or PRM acquisition verification.
when_to_use_negative:
- Input is already converted to an exchange format like mzML (e.g., via ProteoWizard/MSconvert)
  — the index comes directly from the vendor .raw file
- The analysis goal is high-level statistical postprocessing (e.g., MSstats on MaxQuant
  or Skyline outputs) rather than raw scan-level data access
- The data are from a non-Thermo Fisher instrument vendor; rawR wraps the RawFileReader
  API specific to Thermo .raw files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: rawrr
  role: Provides readIndex() and readSpectrum() for enumerating scans and extracting
    spectra directly from Thermo .raw files via the rawrr.exe .NET wrapper
  repo: https://github.com/fgcz/rawrr
- name: rawR
  role: Manuscript package providing OS-independent access to all spectral data logged
    in Thermo Fisher Scientific raw files
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET assemblies (C#) that rawrr wraps to read header, chromatographic,
    and spectral data from Thermo RAW files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative Spectra backend exposing rawrr accessors as an on-disk MsBackend
    for Bioconductor Spectra
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:214e5e228aa17f7f29a59883fdf13a8c690c368d6d341095198c42254f707cc8
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# raw-file-scan-index-extraction

## Summary

Enumerates and filters scans directly from a Thermo Fisher .raw file in R using rawrr::readIndex, without converting to exchange formats like mzML, as the prerequisite step for per-scan spectral extraction or PRM cycle verification.

## When to use

Use when the input is a Thermo Fisher Scientific Orbitrap .raw file (e.g., 20181113_010_autoQC01.raw) and you must enumerate, filter, or select scans (MS1-only or targeted MS2) before per-scan spectral extraction or PRM acquisition verification.

## When NOT to use

- Input is already converted to an exchange format like mzML (e.g., via ProteoWizard/MSconvert) — the index comes directly from the vendor .raw file
- The analysis goal is high-level statistical postprocessing (e.g., MSstats on MaxQuant or Skyline outputs) rather than raw scan-level data access
- The data are from a non-Thermo Fisher instrument vendor; rawR wraps the RawFileReader API specific to Thermo .raw files

## Inputs

- Thermo Fisher Scientific raw file (e.g., 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- Filter criteria such as MS-order 'Ms' for MS1-only scans or a targeted filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'

## Outputs

- data.frame scan index (scan numbers, scan types, filter strings) from rawrr::readIndex
- Two-column table of targeted scan numbers and deltas (scan_deltas.csv) for PRM cycle verification
- Per-scan base-peak table with m/z, intensity, and StartTime (ms1_base_peaks.csv)

## How to apply

Call rawrr::readIndex on the raw file; it invokes the compiled rawrr.exe .NET wrapper (built on Thermo's RawFileReader assembly) via base::system2 and returns a data.frame indexing all scans with scan numbers, scan types, and filter strings. Subset the data.frame to the scans of interest: for MS1-only selection filter on the MS-order 'Ms' (the convention rawR uses for chromatogram filters, filter = 'ms'); for targeted PRM scans select those matching the targeted filter string such as 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'. To confirm PRM acquisition, sort the targeted MS2 scan numbers and compute diff(); a constant delta of 22 scans between consecutive scans means one PRM cycle was maintained. The indexed subset then drives per-scan extraction (rawrr::readSpectrum, centroided 'c' spectra) and downstream tables such as ms1_base_peaks.csv or scan_deltas.csv; the article returns extracted data from the C# layer to R via file I/O. Example raw data for testing can be fetched via ExperimentHub (record EH4547) from the Bioconductor data package tartare or MassIVE dataset MSV000086542.

## Related tools

- **rawrr** (Provides readIndex() and readSpectrum() for enumerating scans and extracting spectra directly from Thermo .raw files via the rawrr.exe .NET wrapper) — https://github.com/fgcz/rawrr
- **rawR** (Manuscript package providing OS-independent access to all spectral data logged in Thermo Fisher Scientific raw files) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET assemblies (C#) that rawrr wraps to read header, chromatographic, and spectral data from Thermo RAW files) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative Spectra backend exposing rawrr accessors as an on-disk MsBackend for Bioconductor Spectra) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
rawfile <- normalizePath(eh[["EH4547"]]); idx <- rawrr::readIndex(rawfile); ms1 <- idx[idx$MSOrder == 'Ms', ]
```

## Evaluation signals

- scan_deltas.csv exists and contains scan-number and delta columns with every recorded delta equal to 22 (constant PRM acquisition cycle)
- ms1_base_peaks.csv row count equals the number of MS1-level scans in the readIndex scan index
- Every row of the base-peak table contains non-empty mZ and intensity values for the base peak
- The subset of the readIndex data.frame matches the target filter string (e.g., 'FTMS + c NSI Full ms2 487.2567@hcd27.00') and scan types are correctly classified by MS order ('Ms' for MS1)
- Raw file provenance is traceable (e.g., USI mzspec:MSV000086542:20181113_010_autoQC01)

## Limitations

- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths
- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- Parallel computation support is a planned next step, not present; exchangeable Spectra backends (MsBackendRawFileReader) were only planned, not yet fully implemented

## Evidence
