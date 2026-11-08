---
name: mass-spectrometry-table-export
description: Use when you need per-scan base peak intensity and m/z across MS1-level
  scans of a Thermo raw file, e.g. to build a base-peak table for instrument QC or
  TIC-like diagnostics from rawR-readable Orbitrap data.
when_to_use_negative:
- When the input is already a feature table, peak list, or vendor-neutral format such
  as mzML — this skill targets proprietary Thermo raw files via the RawFileReader
  API.
- When you need fragment-ion or MS2-level spectra rather than MS1-level base peak
  summaries — use the full readSpectrum() spectral extraction workflow instead.
- When a headless/non-R pipeline is required and no R runtime or .NET (rawrr.exe)
  wrapper is available.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: R package that wraps Thermo RawFileReader to read spectral data, scan index
    and chromatograms from proprietary .raw files; used here via readSpectrum() and
    readIndex() to build the base-peak table.
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET assemblies (compiled C# wrapper rawrr.exe) that rawrr
    invokes to access spectral data logged in Thermo Fisher Scientific raw files.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: ExperimentHub
  role: Bioconductor resource used to fetch the example raw file (EH4547) for the
    base-peak export workflow.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:8886c1dd530b227bfd79b5e3f598939d65295d17032411549287cb1e27300fac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - Bioconductor
  - ExperimentHub
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mass-spectrometry-table-export

## Summary

Extract per-scan spectral attributes (scan number, retention time, base peak m/z and intensity) from Thermo Fisher Scientific raw files using the rawR/rawrr R package and export them as a structured table (e.g. basepeak_ms1.csv) for downstream MS1-level quality and QC analysis.

## When to use

Use when you need per-scan base peak intensity and m/z across MS1-level scans of a Thermo raw file, e.g. to build a base-peak table for instrument QC or TIC-like diagnostics from rawR-readable Orbitrap data.

## When NOT to use

- When the input is already a feature table, peak list, or vendor-neutral format such as mzML — this skill targets proprietary Thermo raw files via the RawFileReader API.
- When you need fragment-ion or MS2-level spectra rather than MS1-level base peak summaries — use the full readSpectrum() spectral extraction workflow instead.
- When a headless/non-R pipeline is required and no R runtime or .NET (rawrr.exe) wrapper is available.

## Inputs

- Thermo Fisher Scientific raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- R environment with the rawrr package installed (wraps Thermo RawFileReader .NET assemblies)

## Outputs

- CSV table (basepeak_ms1.csv) with columns scan, StartTime, mZ, intensity — one row per MS1-level scan
- Per-scan rawrrSpectrum objects / rawrrSpectrumSet with centroided peak lists

## How to apply

Obtain the example raw file 20181113_010_autoQC01.raw via Bioconductor ExperimentHub resource EH4547 or the MassIVE dataset MSV000086542 and verify its MD5 checksum a1f5df9627cf9e0d51ec1906776957ab. Call rawrr::readSpectrum() on the raw file, restricting to MS1-level scans with the filter 'ms'; the call invokes the compiled C# wrapper (rawrr.exe / Thermo RawFileReader) and returns a rawrrSpectrumSet. For each centroided MS1 spectrum, identify the peak with the maximum signal intensity and record its m/z and intensity values. Retrieve the Scan Start Time (StartTime / acquisition StartTime metadata key) for each scan. Assemble the per-scan results into a data frame with columns scan, StartTime, mZ, intensity and write it to basepeak_ms1.csv; validate that the table contains exactly one row per MS1-level scan and that each row's intensity equals the maximum signal intensity of the corresponding spectrum. Alternatively, readIndex() provides a scan index with rtinseconds and MSOrder for filtering scans by MS order.

## Related tools

- **rawrr** (R package that wraps Thermo RawFileReader to read spectral data, scan index and chromatograms from proprietary .raw files; used here via readSpectrum() and readIndex() to build the base-peak table.) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET assemblies (compiled C# wrapper rawrr.exe) that rawrr invokes to access spectral data logged in Thermo Fisher Scientific raw files.) — https://github.com/thermofisherlsms/RawFileReader
- **ExperimentHub** (Bioconductor resource used to fetch the example raw file (EH4547) for the base-peak export workflow.)

## Examples

```
eh <- ExperimentHub::ExperimentHub(); EH4547 <- normalizePath(eh[["EH4547"]]); S <- rawrr::readSpectrum(EH4547, filter = "ms"); write.csv(data.frame(scan=S$scan, StartTime=S$StartTime, mZ=S$basePeakMass, intensity=S$basePeakIntensity), "basepeak_ms1.csv")
```

## Evaluation signals

- The exported basepeak_ms1.csv contains exactly one row per MS1-level scan present in the raw file.
- Each row's intensity equals the maximum signal intensity of the corresponding spectrum (peak with the largest signal intensity).
- Each row's mZ corresponds to the base peak mass of that scan, and StartTime matches the Scan Start Time metadata key.
- The MD5 checksum of the source raw file equals a1f5df9627cf9e0d51ec1906776957ab (for the reference example file).
- Total row count is consistent with the number of MS1 scans reported by the file header (e.g. the reference run produced 2.1881 × 10^4 scans).

## Limitations

- rawR/rawrr requires the .NET-based rawrr.exe wrapper (Thermo RawFileReader); environments without .NET support cannot execute this workflow.
- At the time of publication rawR was not yet submitted to Bioconductor and lacked exchangeable backends and parallel computation support.
- The workflow is specific to Thermo Fisher Scientific Orbitrap raw files; other vendor formats are not supported by this path.

## Evidence

- [results] readSpectrum() returns rawrrSpectrum objects whose summary exposes Scan Start Time, Scan Number, Base Peak Intensity and Base Peak Mass for an individual scan: [evidence span withheld — anchor status: not_found]
- [results] readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms"): "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [methods] The example file `20181113_010_autoQC01.raw` used throughout this manuscript: "The example file `20181113_010_autoQC01.raw` used throughout this manuscript"
- [methods] The file is part of the MassIVE dataset [MSV000086542]: "The file is part of the MassIVE dataset [MSV000086542]"
- [abstract] Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files.: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [results] Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet.: "Individual scans or scan collections (sets) can be read by the function readSpectrum() which returns a rawrrSpectrum object or rawrrSpectrumSet."
- [readme] The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package"
