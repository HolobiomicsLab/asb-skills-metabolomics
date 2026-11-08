---
name: instrument-acquisition-benchmarking
description: Use when comparing acquisition controllers on real LC-MS/MS runs and
  you need to quantify MS2 efficiency (peaks fragmented / MS2 scans), scan counts,
  or per-cycle MS1 scan-plus-processing overhead from mzML files.
when_to_use_negative:
- Data is simulated in ViMMS rather than instrument-acquired mzML — timing measurements
  like the 0.68 s SmartROI overhead only apply to real hardware runs via IAPI
- Input is already a feature/peak table without scan-level mzML data, so inter-scan
  start-time differences cannot be computed
- Goal is comparing fragmentation spectra quality (e.g., via SIRIUS4/CFM-ID) rather
  than acquisition-level counts, efficiency, and timing metrics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and optimize the
    DDA controllers (full-scan, TopN, SmartROI, WeightedDEW) before instrument benchmarking
- name: IAPI
  role: Instrument Application Programming Interface used to deploy the controllers
    on real samples so their timings and efficiency can be measured from acquired
    mzML
- name: MZmine2
  role: Peak picking of full-scan mzML files to obtain bounding boxes against which
    MS2 precursor events are matched to compute efficiency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:c5552e35c38ba1966a9eafec02ae121b9a8314ef80327054a2a29789992ccfcf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine2
  - ViMMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# instrument-acquisition-benchmarking

## Summary

Benchmarks DDA controller strategies (full-scan, TopN, SmartROI, WeightedDEW) by comparing MS1/MS2 scan counts, fragmentation efficiency, and MS1 scan-plus-processing timings from instrument-acquired mzML files, as done in the ViMMS DDA strategy validation on serum (QCA) and beer (QCB) extracts.

## When to use

Use when comparing acquisition controllers on real LC-MS/MS runs and you need to quantify MS2 efficiency (peaks fragmented / MS2 scans), scan counts, or per-cycle MS1 scan-plus-processing overhead from mzML files.

## When NOT to use

- Data is simulated in ViMMS rather than instrument-acquired mzML — timing measurements like the 0.68 s SmartROI overhead only apply to real hardware runs via IAPI
- Input is already a feature/peak table without scan-level mzML data, so inter-scan start-time differences cannot be computed
- Goal is comparing fragmentation spectra quality (e.g., via SIRIUS4/CFM-ID) rather than acquisition-level counts, efficiency, and timing metrics

## Inputs

- Six mzML files per sample (full-scan, TopN, SmartROI, WeightedDEW controllers) for the serum (QCA) and beer (QCB) extracts
- MZmine2 peak-picking output (peak bounding boxes) derived from the full-scan mzML files

## Outputs

- acquisition_scan_metrics.csv with per-controller MS1/MS2 scan counts, fragmentation efficiency (0–1), and MS1 scan-plus-processing timings
- A table mirroring Table 2 and Table SI-6 of the paper

## How to apply

Load the six mzML files per sample (full-scan, TopN, SmartROI, WeightedDEW) for the serum (QCA) and beer (QCB) extracts using pyteomics mzML parsing, and count MS1 and MS2 scans per controller. Pick peaks from the full-scan mzML files with MZmine2 bounding boxes, then match MS2 precursor fragmentation events to those bounding boxes to compute coverage and efficiency (picked peaks fragmented divided by number of MS2 scans). Compute inter-scan start-time differences between successive scans in each mzML file to derive MS1 scan-plus-processing timings per controller (e.g., 0.68 s for SmartROI vs 0.54 s for full-scan, 0.56 s TopN, 0.61 s WeightedDEW on beer). Assemble counts, efficiency, and timing values into a table mirroring Table 2 and Table SI-6 and save as acquisition_scan_metrics.csv; the longer SmartROI timing reflects its real-time ROI tracking overhead.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and optimize the DDA controllers (full-scan, TopN, SmartROI, WeightedDEW) before instrument benchmarking)
- **IAPI** (Instrument Application Programming Interface used to deploy the controllers on real samples so their timings and efficiency can be measured from acquired mzML)
- **MZmine2** (Peak picking of full-scan mzML files to obtain bounding boxes against which MS2 precursor events are matched to compute efficiency)

## Examples

```
from pyteomics import mzml; scans = list(mzml.read('beer_QCB_SmartROI.mzml'))
```

## Evaluation signals

- Beer-extract timings reproduce the reported ordering: SmartROI 0.68 s longest, full-scan 0.54 s shortest, with TopN 0.56 s and WeightedDEW 0.61 s in between
- All efficiency values fall between 0 and 1, with SmartROI highest for the beer sample
- Counts, efficiency, and timing entries are present for all controllers on both samples (QCA and QCB)
- Computed timing values match the SI-reported trends within rounding
- Output CSV schema matches Table 2 / Table SI-6 columns

## Limitations

- Timing differences are attributable to controller processing overhead (e.g., SmartROI's real-time ROI tracking), so comparisons are only meaningful for the same instrument and sample matrix
- Efficiency depends on MZmine2 peak-picking of the full-scan files; picking parameters affect the matched bounding boxes and thus the efficiency estimate
- Results are validated on only two complex samples (serum QCA and beer QCB extracts); generalization to other matrices is not established here

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] on real samples via an Instrument Application Programming Interface (IAPI): "on real samples via an Instrument Application Programming Interface (IAPI)"
- [intro] Finally, we describe two new DDA strategies prototyped in this way and demonstrate, through validation on two complex samples, their improvement over traditional DDA approaches.: "Finally, we describe two new DDA strategies prototyped in this way and demonstrate, through validation on two complex samples, their improvement over traditional DDA approaches."
