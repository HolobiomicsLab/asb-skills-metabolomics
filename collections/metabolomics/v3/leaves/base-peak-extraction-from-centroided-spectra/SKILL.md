---
name: base-peak-extraction-from-centroided-spectra
description: Use when you hold a Thermo Fisher .raw file, need per-MS1-scan base peak
  m/z and intensity, and want direct in-R access without ProteoWizard/mzML conversion.
when_to_use_negative:
- Data has already been converted to mzML or another exchange format — an mzML-based
  reader is the appropriate choice then
- The analysis needs MS2 fragment spectra or fragment-ion traces (e.g. 'FTMS + c NSI
  Full ms2 487.2567@hcd27.00' filters) rather than MS1 base peaks
- Input is a preprocessed feature/peak table rather than raw instrument data
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: 'Primary R interface: readIndex() enumerates scans, readSpectrum() loads centroided
    (c) per-scan spectra from the raw file'
  repo: https://github.com/fgcz/rawrr
- name: rawR
  role: Predecessor package using the same scan-index filtering convention (filter
    = 'ms') for MS1-only selection
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Thermo Fisher vendor .NET API that rawrr wraps to access spectral data logged
    in proprietary raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:e07d99f0e7b2b12b57eb8f6f3296b68bdcdb6751f08cab907c24ca659b193c6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# base-peak-extraction-from-centroided-spectra

## Summary

Extracts the base peak (m/z and intensity at maximum signal) from each centroided MS1 spectrum in a Thermo Fisher raw file via rawrr, producing a per-scan table for QC-style monitoring without converting data to mzML.

## When to use

Use when you hold a Thermo Fisher .raw file, need per-MS1-scan base peak m/z and intensity, and want direct in-R access without ProteoWizard/mzML conversion.

## When NOT to use

- Data has already been converted to mzML or another exchange format — an mzML-based reader is the appropriate choice then
- The analysis needs MS2 fragment spectra or fragment-ion traces (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00' filters) rather than MS1 base peaks
- Input is a preprocessed feature/peak table rather than raw instrument data

## Inputs

- Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542)
- rawrr::readIndex() scan index data.frame

## Outputs

- Data frame with one row per MS1 scan: base peak m/z, base peak intensity, scan StartTime
- ms1_base_peaks.csv

## How to apply

Enumerate all scans with rawrr::readIndex(), which invokes the compiled rawR.exe .NET wrapper via base::system2, then subset the returned data.frame to MS1-level scans by filtering the MS-order on 'Ms' (the same MS1-only convention rawR uses for chromatogram filters, filter = 'ms'). For each selected scan, load the centroided spectrum with rawrr::readSpectrum using the centroiding option (c) described in the article, then identify the position (m/z) and intensity pair at the maximum signal intensity and record the scan's StartTime. Combine one row per MS1 scan into a data frame and write ms1_base_peaks.csv. Rationale: rawR wraps the vendor RawFileReader .NET API so spectra are read directly from the proprietary raw file, avoiding the exchange-format detour through mzML. Validation: the row count must equal the number of MS1-level scans in the readIndex() output, and every row must carry non-empty mZ and intensity values for the base peak.

## Related tools

- **rawrr** (Primary R interface: readIndex() enumerates scans, readSpectrum() loads centroided (c) per-scan spectra from the raw file) — https://github.com/fgcz/rawrr
- **rawR** (Predecessor package using the same scan-index filtering convention (filter = 'ms') for MS1-only selection) — https://github.com/fgcz/rawR
- **RawFileReader** (Thermo Fisher vendor .NET API that rawrr wraps to access spectral data logged in proprietary raw files) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
idx <- rawrr::readIndex(rawfile); ms1 <- idx[idx$MSOrder == 'Ms', ]; S <- rawrr::readSpectrum(rawfile, ms1$scan, mode='c'); bp <- lapply(S, function(s) data.frame(scan=s$scan, StartTime=s$StartTime, mZ=s$mZ[which.max(s$intensity)], intensity=max(s$intensity)))
```

## Evaluation signals

- Output row count equals the number of MS1-level scans found in the rawrr::readIndex() scan index
- Every output row contains non-empty mZ and intensity values for the base peak
- Base peak is the (m/z, intensity) pair at the maximum signal intensity within each centroided scan
- Each row carries a valid StartTime for its scan
- On Windows, mZ and intensity vectors parse with correct lengths only if the decimal symbol is configured as '.'

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows the decimal symbol must be configured as '.' or the mZ and intensity vectors get wrong lengths
- Exchangeable Spectra backends (MsBackendRawFileReader) and parallel computation were planned but not yet implemented at writing time

## Evidence

- [results] MS1-only selection convention rawR uses for chromatogram filters: "Each XIC was calculated using a tolerance of 10 ppm around the target mass and using only MS1-level scans."
- [methods] Centroiding option used when writing spectra to file: "All spectra were written to file after applying centroiding (c) and lock mass correction"
- [abstract] rawR provides direct access to raw spectral data: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files"
- [intro] rawrr wraps vendor RawFileReader API: "utilizes a vendor-provided API named `RawFileReader` [@rawfilereader] to access spectral data logged in proprietary Thermo Fisher Scientific raw files"
- [readme] rawrr provides access to proprietary Thermo instrument data: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package"
