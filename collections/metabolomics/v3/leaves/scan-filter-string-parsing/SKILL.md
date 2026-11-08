---
name: scan-filter-string-parsing
description: Use when a Thermo Fisher .raw file has been indexed via rawrr::readIndex
  and you need to classify scans by their filter string — e.g. to isolate targeted
  MS2 scans for a precursor like 487.2567@hcd27.00 and compute inter-scan deltas.
when_to_use_negative:
- The data are already de-identified spectra or an mzML file with no Thermo filter
  strings to parse.
- The goal is centroiding, lock-mass correction, or raw-spectrum extraction rather
  than scan classification — use rawrr::readSpectrum or similar instead.
- Only a summary chromatogram or XIC is needed and scan-level index detail is irrelevant.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Provides readIndex() that wraps the RawFileReader .NET assembly to return
    scan numbers, scan types, and filter strings per scan; used to extract the index
    that this skill parses.
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Thermo Fisher .NET assemblies invoked by rawrr.exe to read the proprietary
    .raw file and produce the scan metadata (including filter strings) consumed by
    this skill.
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:1963e652891f900884efad2748eb170798afffdde7d92ed6e427ece2caae9d61
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# scan-filter-string-parsing

## Summary

Parses Thermo Fisher Scientific filter strings (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]') attached to each scan record returned by rawrr::readIndex, to identify targeted MS2 scans and characterize acquisition cycles in a PRM method.

## When to use

Use when a Thermo Fisher .raw file has been indexed via rawrr::readIndex and you need to classify scans by their filter string — e.g. to isolate targeted MS2 scans for a precursor like 487.2567@hcd27.00 and compute inter-scan deltas.

## When NOT to use

- The data are already de-identified spectra or an mzML file with no Thermo filter strings to parse.
- The goal is centroiding, lock-mass correction, or raw-spectrum extraction rather than scan classification — use rawrr::readSpectrum or similar instead.
- Only a summary chromatogram or XIC is needed and scan-level index detail is irrelevant.

## Inputs

- Thermo Fisher Scientific .raw instrument file (e.g. 20181113_010_autoQC01.raw)
- Scan index from rawrr::readIndex (scan number, scan type, filter string columns)

## Outputs

- Vector of scan numbers whose filter string matches the targeted MS2 method
- Vector of inter-scan deltas (scan number differences) for the targeted scans
- Two-column table (scan number, delta) written to scan_deltas.csv

## How to apply

Load the .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or Bioconductor tartare EH4547) and call rawrr::readIndex to obtain scan numbers, scan types, and filter strings via the RawFileReader wrapper. Select the scans whose filter string matches the targeted MS2 method — in the article's example, scans with filter 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'. Sort the matching scan numbers and apply diff() to compute the delta between consecutive targeted scans; a constant delta (22 in the article's PRM example) confirms one PRM acquisition cycle per targeted event. Deviations from the expected delta flag irregular acquisition. The rationale is that filter strings encode precursor m/z, collision energy (hcd27.00), and scan window, making them a reliable key for isolating the targeted-MS2 subpopulation without re-scanning the file for raw spectra.

## Related tools

- **rawrr** (Provides readIndex() that wraps the RawFileReader .NET assembly to return scan numbers, scan types, and filter strings per scan; used to extract the index that this skill parses.) — https://github.com/fgcz/rawrr
- **RawFileReader** (Thermo Fisher .NET assemblies invoked by rawrr.exe to read the proprietary .raw file and produce the scan metadata (including filter strings) consumed by this skill.) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
R -e 'library(rawrr); idx <- rawrr::readIndex("20181113_010_autoQC01.raw"); ms2 <- idx[grepl("Full ms2 487.2567", idx$filter), ]; d <- diff(sort(ms2$scanNumber)); write.csv(data.frame(scanNumber=ms2$scanNumber[-1], delta=d), "scan_deltas.csv", row.names=FALSE)'
```

## Evaluation signals

- scan_deltas.csv exists and contains scan-number and delta columns.
- All recorded deltas equal the expected PRM cycle length (22 in the article's example).
- Every delta is a positive integer (no negative or zero gaps) — sorted scan numbers imply monotonic deltas.
- The subset of scans matching the targeted filter string is non-empty and consistent with the number of targeted events declared in the method.

## Limitations

- rawrrSpectrum and index content partially depends on the instrument model and installed instrument control software version.
- The rawrr.exe wrapper must be installed (rawrr::installRawrrExe()) and file I/O is used between the R and C# layers.
- On Windows, the decimal symbol must be configured as '.' for m/z and intensity vectors to have correct lengths.

## Evidence

- [methods] readIndex returns scan numbers, scan types, and filter strings: [evidence span withheld — anchor status: not_found]
- [results] Constant delta of 22 scans confirms PRM: [evidence span withheld — anchor status: not_found]
