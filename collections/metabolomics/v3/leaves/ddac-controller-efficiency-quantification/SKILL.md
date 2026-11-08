---
name: ddac-controller-efficiency-quantification
description: Use when you have mzML files from multiple DDA controllers run on the
  same sample (e.g. serum/beer extracts) and need to compare scan counts, fragmentation
  efficiency, or MS1 cycle overhead to validate an acquisition strategy.
when_to_use_negative:
- Input data are simulated ViMMS runs rather than real-instrument mzML files — timings
  derived from inter-scan start-time differences validate controller overhead on real
  hardware, not in silico results
- You only need theoretical upper bounds on DDA coverage for an injection rather than
  empirical efficiency of specific controllers
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: virtual metabolomics mass spectrometer used to prototype, optimize and simulate
    the DDA controllers before real-instrument validation
- name: IAPI
  role: Instrument Application Programming Interface used to run the DDA controllers
    on real samples and record acquisitions
- name: MZmine2
  role: peak picking to produce bounding boxes from full-scan mzML files, against
    which MS2 fragmentation events are matched to compute coverage/efficiency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:b70033530488a64ec8a1a62780882e5cc7330493afb1182716a251ac5736a920
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine2
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ddac-controller-efficiency-quantification

## Summary

Quantifies and compares DDA controller performance (full-scan, TopN, SmartROI, WeightedDEW) on real LC-MS/MS acquisitions by counting MS1/MS2 scans in mzML files, computing fragmentation efficiency (picked peaks fragmented / MS2 scans) via MZmine2 peak bounding boxes, and deriving per-controller MS1 scan-plus-processing timings from inter-scan start-time differences.

## When to use

Use when you have mzML files from multiple DDA controllers run on the same sample (e.g. serum/beer extracts) and need to compare scan counts, fragmentation efficiency, or MS1 cycle overhead to validate an acquisition strategy.

## When NOT to use

- Input data are simulated ViMMS runs rather than real-instrument mzML files — timings derived from inter-scan start-time differences validate controller overhead on real hardware, not in silico results
- You only need theoretical upper bounds on DDA coverage for an injection rather than empirical efficiency of specific controllers

## Inputs

- mzML files of DDA acquisitions per controller (full-scan, TopN, SmartROI, WeightedDEW) on the same sample extract (e.g. six mzML files per sample for serum QCA and beer QCB)
- MZmine2 peak-picking output / bounding boxes from the full-scan mzML files
- MS2 precursor information (precursor m/z, retention time) extracted from the mzML files

## Outputs

- acquisition_scan_metrics.csv: table of MS1/MS2 scan counts, fragmentation efficiency, and MS1 scan-plus-processing timings per controller per sample (mirroring Table 2 and Table SI-6)

## How to apply

Load the six mzML files per sample (full-scan, TopN, SmartROI shifts, WeightedDEW for serum QCA and beer QCB) and parse them with pyteomics/mzml to count MS1 and MS2 scans per controller. Pick peaks from the full-scan mzML files using MZmine2 bounding boxes, then match MS2 precursor fragmentation events to those bounding boxes to compute coverage, defining efficiency as picked peaks fragmented divided by the number of MS2 scans (values between 0 and 1, SmartROI expected highest for beer). Compute inter-scan start-time differences of successive scans to obtain MS1 scan-plus-processing timings per controller, where SmartROI is expected longest (0.68 s on beer extract) due to real-time ROI tracking overhead, versus full-scan shortest (0.54 s), TopN (0.56 s), and WeightedDEW (0.61 s). Assemble counts, efficiency, and timing values into a table mirroring Table 2 and Table SI-6 and save as acquisition_scan_metrics.csv.

## Related tools

- **ViMMS** (virtual metabolomics mass spectrometer used to prototype, optimize and simulate the DDA controllers before real-instrument validation)
- **IAPI** (Instrument Application Programming Interface used to run the DDA controllers on real samples and record acquisitions)
- **MZmine2** (peak picking to produce bounding boxes from full-scan mzML files, against which MS2 fragmentation events are matched to compute coverage/efficiency)

## Examples

```
from pyteomics import mzml; spectra = list(mzml.read('QCB_beer_SmartROI.mzML')); ms1 = [s for s in spectra if s.get('ms level') == 1]
```

## Evaluation signals

- Beer-extract timings reproduce the reported ordering: SmartROI 0.68 s longest, full-scan 0.54 s shortest, TopN 0.56 s, WeightedDEW 0.61 s
- All efficiency values fall between 0 and 1, with SmartROI highest for the beer (QCB) sample
- Counts, efficiency, and timing entries are present for all five controllers on both samples (QCA serum and QCB beer)
- Computed timing values match the SI-reported trends within rounding
- Output CSV schema mirrors Table 2 and Table SI-6 of the paper

## Limitations

- Fragmentation efficiency depends on MZmine2 peak-picking quality in the full-scan reference files; errors in bounding boxes propagate into coverage estimates
- MS1 scan-plus-processing timing is inferred from inter-scan start-time differences, so it conflates scan time with processing overhead and is sensitive to cycle-level variability
- Validated on only two complex samples (serum QCA and beer QCB extracts); results may not transfer to other matrices or instrument configurations

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset"
- [abstract] real-sample validation via IAPI: "on real samples via an Instrument Application Programming Interface (IAPI)"
- [intro] ViMMS prototyping and validation: "optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),"
- [abstract] efficiency context: "development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies."
