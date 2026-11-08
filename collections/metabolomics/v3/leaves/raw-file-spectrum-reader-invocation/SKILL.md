---
name: raw-file-spectrum-reader-invocation
description: Use when analysis needs spectral or chromatographic data read directly
  from a proprietary Thermo Orbitrap .raw file in R, without prior mzML conversion
  via ProteoWizard/MSconvert.
when_to_use_negative:
- The input is already converted to an exchange format like mzML — the paper notes
  such conversion is primarily supported by ProteoWizard/MSconvert and other R packages
  handle that path.
- The workflow requires an exchangeable Spectra backend or parallel data access —
  the article states these are planned next steps, not implemented in rawR.
- Running on Windows with the decimal symbol configured to something other than '.'
  — mZ and intensity vectors will have incorrect lengths.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: rawR/rawrr
  role: R package providing readSpectrum/readChromatogram that wrap RawFileReader
    to build rawrrSpectrum/rawrrChromatogram objects
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor .NET assembly (RawFileReader API) invoked by rawrr.exe to read header,
    chromatographic, and spectral data from .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: tartare
  role: Bioconductor data package supplying the example raw file (EH4547) for demonstration
    and extended testing
- name: ExperimentHub
  role: Fetches the example raw file resource EH4547 locally
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:a75e5341d7d972938f85d74ca3d14acc92eec2d978b894a1b2662891142dc602
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - ExperimentHub
  - R
  - RawFileReader
  - rawR
  - rawR/rawrr
  - tartare
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# raw-file-spectrum-reader-invocation

## Summary

Constructs non-standard R objects (rawrrSpectrum/rawrrChromatogram and their *Set collections) directly from Thermo Fisher Scientific .raw files by invoking rawrr::readSpectrum() and rawrr::readChromatogram(), which wrap the vendor RawFileReader .NET API via the rawrr.exe runtime.

## When to use

Use when analysis needs spectral or chromatographic data read directly from a proprietary Thermo Orbitrap .raw file in R, without prior mzML conversion via ProteoWizard/MSconvert.

## When NOT to use

- The input is already converted to an exchange format like mzML — the paper notes such conversion is primarily supported by ProteoWizard/MSconvert and other R packages handle that path.
- The workflow requires an exchangeable Spectra backend or parallel data access — the article states these are planned next steps, not implemented in rawR.
- Running on Windows with the decimal symbol configured to something other than '.' — mZ and intensity vectors will have incorrect lengths.

## Inputs

- Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- Scan index of interest (e.g. 9594)
- Target m/z list with tolerance (10 ppm) and filter string ('ms') for XIC extraction
- Installed rawrr.exe runtime (rawrr::installRawrrExe())

## Outputs

- rawrrSpectrumSet / rawrrSpectrum object (mZ, intensity vectors, scan header fields)
- rawrrChromatogramSet / rawrrChromatogram object (XIC traces)
- Validation results from is.rawrrSpectrum / is.rawrrChromatogram / validate_rawrrSpectrum

## How to apply

First install the .NET runtime executable with rawrr::installRawrrExe(), since rawrr wraps RawFileReader and exchanges data with the C# layer via file I/O. Obtain the example raw file 20181113_010_autoQC01.raw from MassIVE dataset MSV000086542 or via ExperimentHub resource EH4547 (tartare package), verifying its MD5 checksum a1f5df9627cf9e0d51ec1906776957ab. Call rawrr::readSpectrum() on a specific scan (e.g. scan 9594, assigned to the doubly-charged iRT peptide LGGNEQVTR/2) to receive a rawrrSpectrumSet, and rawrr::readChromatogram() with type 'xic', a 10 ppm tolerance around target masses, and filter 'ms' to restrict to MS1-level scans, receiving a rawrrChromatogramSet. Confirm correctness with is.rawrrSpectrum, is.rawrrChromatogram, and validate_rawrrSpectrum on the returned instances; note that rawrrSpectrum content partially depends on the instrument model and installed instrument control software version.

## Related tools

- **rawR/rawrr** (R package providing readSpectrum/readChromatogram that wrap RawFileReader to build rawrrSpectrum/rawrrChromatogram objects) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor .NET assembly (RawFileReader API) invoked by rawrr.exe to read header, chromatographic, and spectral data from .raw files) — https://github.com/thermofisherlsms/RawFileReader
- **tartare** (Bioconductor data package supplying the example raw file (EH4547) for demonstration and extended testing)
- **ExperimentHub** (Fetches the example raw file resource EH4547 locally)

## Examples

```
library(ExperimentHub); eh <- ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(rawfile, 9594); is.rawrrSpectrum(S[[1]])
```

## Evaluation signals

- class(S) returns 'rawrrSpectrumSet' (or is.rawrrSpectrum TRUE on the extracted element) after readSpectrum(rawfile, 9594)
- class(C) returns 'rawrrChromatogramSet' (is.rawrrChromatogram TRUE) after readChromatogram with type 'xic'
- validate_rawrrSpectrum() completes without error on the constructed spectrum object
- Downloaded file MD5 equals a1f5df9627cf9e0d51ec1906776957ab
- XIC traces contain only MS1-level scans (filter 'ms') at 10 ppm tolerance, and scan 9594 resolves to peptide LGGNEQVTR/2 per the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' or the mZ and intensity vectors will have incorrect lengths
- Exchangeable Spectra backends (MsBackendRawFileReader) and parallel computation support are planned next steps, not yet complete
- Data exchange between the R and C# layers occurs via file I/O, adding overhead

## Evidence
