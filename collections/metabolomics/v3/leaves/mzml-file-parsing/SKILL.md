---
name: mzml-file-parsing
description: Use when raw acquisition output from DDA controllers (full-scan, TopN,
  SmartROI, WeightedDEW) is available as mzML files and scan counts, precursor events,
  or inter-scan timing metrics must be extracted before efficiency or timing analysis.
when_to_use_negative:
- Input is already an aggregated metrics table (e.g. acquisition_scan_metrics.csv)
  — no mzML parsing needed
- Goal is peak picking or fragmentation coverage itself (MZmine2 bounding boxes matched
  to MS2 precursor events) rather than scan-level extraction
- Data are not in mzML (e.g. raw vendor formats) and no mzML conversion step is available
tools:
- name: pyteomics/mzml
  role: Parses the mzML files to enumerate MS1/MS2 scans and read start-time and precursor
    metadata
- name: ViMMS
  role: Source of in silico mzML acquisitions that can be parsed identically to real-instrument
    data
- name: IAPI
  role: Instrument Application Programming Interface used to generate real-sample
    acquisitions that are then parsed from mzML
- name: MZmine2
  role: Consumes parsed full-scan mzML for peak picking when coverage/efficiency metrics
    are computed downstream
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:a2ecefff28153b31aaf5ad03790488851f76f5a1ce68bcf53c70abdc96a705bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine2
  - ViMMS
  - pyteomics/mzml
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mzml-file-parsing

## Summary

Parse raw LC-MS/MS acquisition files in mzML format (produced by ViMMS simulations or real instruments via IAPI) to enumerate MS1/MS2 scans, extract precursor and start-time metadata, and derive per-controller acquisition metrics such as scan counts and MS1 scan-plus-processing timings.

## When to use

Use when raw acquisition output from DDA controllers (full-scan, TopN, SmartROI, WeightedDEW) is available as mzML files and scan counts, precursor events, or inter-scan timing metrics must be extracted before efficiency or timing analysis.

## When NOT to use

- Input is already an aggregated metrics table (e.g. acquisition_scan_metrics.csv) — no mzML parsing needed
- Goal is peak picking or fragmentation coverage itself (MZmine2 bounding boxes matched to MS2 precursor events) rather than scan-level extraction
- Data are not in mzML (e.g. raw vendor formats) and no mzML conversion step is available

## Inputs

- mzML files for each sample and DDA controller (full-scan, TopN, SmartROI shifts, WeightedDEW), e.g. six mzML files per sample for serum (QCA) and beer (QCB) extracts
- Optional: controller/sample mapping defining which mzML file belongs to which DDA condition

## Outputs

- Per-controller MS1 and MS2 scan counts
- Inter-scan start-time differences yielding MS1 scan-plus-processing timings per controller
- Parsed scan-level records (precursor events, start times) for downstream coverage/efficiency computation
- Rows for acquisition_scan_metrics.csv mirroring Table 2 and Table SI-6

## How to apply

Load the mzML files per sample and controller (e.g. the six files per sample for serum QCA and beer QCB extracts) using pyteomics/mzml parsing. Iterate over the scan list to count MS1 and MS2 scans per controller, and read precursor m/z, intensity, and scan start-time fields from each MS2 event. Compute inter-scan start-time differences for successive scans to obtain per-controller MS1 scan-plus-processing timings (on beer: SmartROI 0.68 s vs full-scan 0.54 s, TopN 0.56 s, WeightedDEW 0.61 s). Decision point: parsed scans can feed peak-picking coverage analysis (MZmine2 bounding boxes matched to MS2 precursor events) or feed direct assembly of an acquisition_scan_metrics.csv table mirroring Table 2 and Table SI-6. Rationale: mzML is the common interchange format shared by ViMMS in silico output and real-instrument runs, so parsing it once enables identical metric computation on both.

## Related tools

- **pyteomics/mzml** (Parses the mzML files to enumerate MS1/MS2 scans and read start-time and precursor metadata)
- **ViMMS** (Source of in silico mzML acquisitions that can be parsed identically to real-instrument data)
- **IAPI** (Instrument Application Programming Interface used to generate real-sample acquisitions that are then parsed from mzML)
- **MZmine2** (Consumes parsed full-scan mzML for peak picking when coverage/efficiency metrics are computed downstream)

## Examples

```
import pyteomics.mzml as mzml; scans = list(mzml.read('QCB_SmartROI.mzML')); ms1 = [s for s in scans if s['ms level'] == 1]; ms2 = [s for s in scans if s['ms level'] == 2]
```

## Evaluation signals

- MS1 and MS2 scan counts are present for all controllers on both samples (QCA serum and QCB beer)
- Beer-extract MS1 timings reproduce the reported ordering: SmartROI 0.68 s longest, full-scan 0.54 s shortest, TopN 0.56 s and WeightedDEW 0.61 s in between
- Derived efficiency values (picked peaks fragmented / number of MS2 scans) fall between 0 and 1, with SmartROI highest for beer
- Computed timing values match the SI-reported trends within rounding
- All counts, efficiency, and timing entries are populated in acquisition_scan_metrics.csv

## Limitations

- MS1 scan-plus-processing timings are approximated from inter-scan start-time differences, so they reflect acquisition-plus-overhead gaps rather than isolated processing cost
- Metrics depend on the controller run having produced complete mzML output (real runs via IAPI); missing or truncated files break the six-files-per-sample assembly
- Timings are comparable across controllers only when parsed from the same sample type (e.g. beer extract values differ from serum values)

## Evidence

- [methods] Load the six mzML files per sample (full-scan, TopN, SmartROI shifts, WeightedDEW) for the serum (QCA) and beer (QCB) extracts using pyteomics/mzml parsing.: [evidence span withheld — anchor status: not_found]
- [methods] Compute inter-scan start-time differences for successive scans in each mzML file to obtain the MS1 scan-plus-processing timings per controller: [evidence span withheld — anchor status: not_found]
- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [abstract] on real samples via an Instrument Application Programming Interface (IAPI): "on real samples via an Instrument Application Programming Interface (IAPI)"
