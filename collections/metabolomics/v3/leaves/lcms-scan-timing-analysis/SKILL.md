---
name: lcms-scan-timing-analysis
description: Use when comparing DDA acquisition controllers' scan overhead — e.g.
  mzML files from full-scan, TopN, SmartROI, and WeightedDEW runs need inter-scan
  start-time deltas converted into MS1 scan-plus-processing timings to verify controller
  processing overhead.
when_to_use_negative:
- Data comes from ViMMS in silico simulations only — simulated cycles lack the real-instrument
  processing overhead (IAPI-validated runs) that this skill is designed to expose
- The analysis goal is fragmentation coverage/efficiency rather than scan-cycle timing
  — efficiency requires MZmine2 peak picking and MS2 precursor-to-bounding-box matching,
  not inter-scan deltas
- Files are DIA or lack per-scan start-time metadata in the mzML, so successive inter-scan
  differences cannot be computed
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and optimize the
    DDA controllers whose scan timing is later measured on real data
- name: IAPI
  role: Instrument Application Programming Interface used to deploy and validate the
    controllers on real samples, producing the mzML whose scan timings this skill
    extracts
- name: MZmine2
  role: Peak picking (bounding boxes) on full-scan mzML files to derive the fragmentation-efficiency
    column that accompanies the timing metrics in the metrics table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:1eec5276f087ed4f888acdc8cc8d8cb2e3ca28d04e137beef92d2ccd1a7de507
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine2
  - ViMMS
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcms-scan-timing-analysis

## Summary

Quantifies per-cycle MS1 scan-plus-processing overhead for competing DDA acquisition controllers by computing inter-scan start-time differences directly from mzML files. It exists to test whether controller-side real-time logic (e.g. SmartROI's ROI tracking) inflates cycle time on a real instrument, complementing fragmentation-efficiency metrics.

## When to use

Use when comparing DDA acquisition controllers' scan overhead — e.g. mzML files from full-scan, TopN, SmartROI, and WeightedDEW runs need inter-scan start-time deltas converted into MS1 scan-plus-processing timings to verify controller processing overhead.

## When NOT to use

- Data comes from ViMMS in silico simulations only — simulated cycles lack the real-instrument processing overhead (IAPI-validated runs) that this skill is designed to expose
- The analysis goal is fragmentation coverage/efficiency rather than scan-cycle timing — efficiency requires MZmine2 peak picking and MS2 precursor-to-bounding-box matching, not inter-scan deltas
- Files are DIA or lack per-scan start-time metadata in the mzML, so successive inter-scan differences cannot be computed

## Inputs

- mzML files, one per acquisition controller (full-scan, TopN, SmartROI, WeightedDEW) per sample (serum QCA, beer QCB extract)
- Controller-to-file mapping labels (which mzML corresponds to which DDA controller/shift)
- Optionally MZmine2 peak bounding boxes from full-scan mzML for the paired efficiency metric

## Outputs

- acquisition_scan_metrics.csv with columns for MS1/MS2 scan counts, fragmentation efficiency (0–1), and MS1 scan-plus-processing timing per controller per sample (mirroring Table 2 and Table SI-6)
- Per-controller per-cycle timing values in seconds (e.g. SmartROI 0.68 s vs full-scan 0.54 s on beer)

## How to apply

Load the mzML files per sample and controller (e.g. the six files per sample for full-scan, TopN, SmartROI shifts, and WeightedDEW on serum QCA and beer QCB extracts) using pyteomics/mzml parsing. For each file, first count MS1 and MS2 scans, then compute inter-scan start-time differences for successive scans; the aggregate of these deltas yields the per-cycle MS1 scan-plus-processing timing per controller. The decision point is the comparison against the reference ordering: on the beer extract, SmartROI should show the longest per-cycle time (0.68 s, reflecting its real-time ROI tracking overhead) versus WeightedDEW 0.61 s, TopN 0.56 s, and full-scan 0.54 s the shortest. Assemble timing values alongside scan counts and fragmentation efficiency (picked peaks fragmented / number of MS2 scans, from MZmine2 peak bounding boxes matched to MS2 precursor events) into acquisition_scan_metrics.csv mirroring Table 2 and Table SI-6. Validate that timings are positive, consistent across controllers, and reproduce the SI-reported ordering within rounding.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and optimize the DDA controllers whose scan timing is later measured on real data)
- **IAPI** (Instrument Application Programming Interface used to deploy and validate the controllers on real samples, producing the mzML whose scan timings this skill extracts)
- **MZmine2** (Peak picking (bounding boxes) on full-scan mzML files to derive the fragmentation-efficiency column that accompanies the timing metrics in the metrics table)

## Examples

```
from pyteomics import mzml; ts=[s['scanList']['scan'][0]['scan start time'] for s in mzml('QCB_SmartROI.mzML')]; d=[b-a for a,b in zip(ts,ts[1:])]; print(sum(d)/len(d))
```

## Evaluation signals

- Beer-extract timing ordering is reproduced: SmartROI 0.68 s (longest) > WeightedDEW 0.61 s > TopN 0.56 s > full-scan 0.54 s (shortest)
- All timing values are positive and within rounding agreement of the SI-reported trends (Table SI-6)
- Counts, efficiency, and timing entries are present for all five controllers on both samples (serum QCA and beer QCB)
- Efficiency values fall in (0, 1) with SmartROI highest for the beer sample
- Inter-scan deltas are computed from successive scans only within the same mzML file (no cross-file joins)

## Limitations

- Timings are instrument- and sample-specific (Thermo real-instrument runs of beer extract QCB / serum QCA via IAPI); the 0.68 s vs 0.54 s overhead may not transfer to other instruments or gradient conditions
- Timing deltas conflate scan time and controller processing; only the aggregate per-cycle value is recoverable from mzML, not a decomposition
- Values compared 'within rounding' — small deviations from the SI-reported numbers are expected and acceptable

## Evidence

- [results] total processing and MS1 scan time was 0.68 s for SmartROI on the beer extract, versus 0.54 s for full-scan, 0.56 s for TopN, and 0.61 s for WeightedDEW, confirming SmartROI's extra processing overhead.: [evidence span withheld — anchor status: not_found]
- [methods] Compute inter-scan start-time differences for successive scans in each mzML file to obtain the MS1 scan-plus-processing timings per controller.: [evidence span withheld — anchor status: not_found]
- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans"
