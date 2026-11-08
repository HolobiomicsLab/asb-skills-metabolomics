---
name: mzml-scan-parsing
description: Use when the input is an mzML file from an LC-MS/MS injection and the
  analysis needs MS2 fragmentation events (precursor m/z + RT) or MS1/MS2 scan schedules,
  e.g. to score DDA controller coverage of picked peaks.
when_to_use_negative:
- The input is already a picked-peak bounding-box table or a coverage flag table rather
  than raw mzML acquisition data
- The goal is peak picking itself — that is done in MZmine 2 (Supporting Information
  Table SI-1 parameters) before the parsed MS2 events are matched to boxes
- The data are not mzML scan-level acquisitions, e.g. structural comparison tasks
  handled by SIRIUS4/CFM-ID or substructure discovery with MS2LDA
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer that consumes and simulates the parsed
    MS1/MS2 scan schedule when prototyping, optimizing, and validating DDA controllers
    in silico
- name: MZmine 2
  role: Performs the peak picking whose RT/m/z bounding boxes the parsed MS2 fragmentation
    events are matched against to define coverage
- name: IAPI
  role: Instrument Application Programming Interface through which the real-sample
    injections producing the mzML files to be parsed were acquired
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:eb31058f74b9153f5b6e892f88fdfb653b566a78eb37d97c47946ac893dd599c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine 2
  - ViMMS
  - XCMS
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mzml-scan-parsing

## Summary

Parse mzML files to extract per-scan metadata (MS1 survey scans and MS2 fragmentation events with retention times and precursor m/z) so that downstream DDA-coverage and ion-prioritization analyses in ViMMS/IAPI workflows can operate on acquisition data.

## When to use

Use when the input is an mzML file from an LC-MS/MS injection and the analysis needs MS2 fragmentation events (precursor m/z + RT) or MS1/MS2 scan schedules, e.g. to score DDA controller coverage of picked peaks.

## When NOT to use

- The input is already a picked-peak bounding-box table or a coverage flag table rather than raw mzML acquisition data
- The goal is peak picking itself — that is done in MZmine 2 (Supporting Information Table SI-1 parameters) before the parsed MS2 events are matched to boxes
- The data are not mzML scan-level acquisitions, e.g. structural comparison tasks handled by SIRIUS4/CFM-ID or substructure discovery with MS2LDA

## Inputs

- mzML file from a full-scan, TopN, SmartROI, WeightedDEW, or optimal-reference LC-MS/MS injection (e.g. serum QCA or beer QCB extracts)
- Picked-peak bounding boxes (m/z and RT min/max) exported from MZmine 2 for the corresponding full-scan run

## Outputs

- Table of MS2 fragmentation events with retention times and precursor m/z
- MS1/MS2 scan schedule with per-ion most-recent fragmentation times tf (for WeightedDEW weighting)
- Per-peak coverage flag table (coverage_flags.csv) with bounding-box coordinates and a boolean coverage flag
- Per-scan WeightedDEW priority ranking table with weight column in [0, 1]

## How to apply

Read the mzML file and extract all MS2 fragmentation events with their retention times (and precursor m/z), together with the MS1 survey-scan schedule needed by controllers such as TopN or WeightedDEW. For coverage analyses, match each parsed MS2 event against picked-peak bounding boxes (RT and m/z min/max, with the RT range set by the first and last MS1 scans comprising the peak as exported from MZmine 2); an event is attributed to every bounding box it falls into, and a peak is covered if it contains at least one fragmentation event. For prioritization analyses such as WeightedDEW, use the parsed MS1/MS2 scan schedule to determine, per MS1 scan and per ion above the minimum MS1 intensity threshold, the most recent fragmentation time tf of its m/z from the prior MS2 schedule before computing the weight w with t0 = 15 s and t1 = 120 s. The parser feeds both the in silico ViMMS pipeline and real-instrument IAPI injections, so parsed events must retain precursor m/z and RT fields sufficient for downstream matching to the six-injection comparison (full-scan, TopN, SmartROI shift 1/2, WeightedDEW, optimal reference).

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer that consumes and simulates the parsed MS1/MS2 scan schedule when prototyping, optimizing, and validating DDA controllers in silico)
- **MZmine 2** (Performs the peak picking whose RT/m/z bounding boxes the parsed MS2 fragmentation events are matched against to define coverage)
- **IAPI** (Instrument Application Programming Interface through which the real-sample injections producing the mzML files to be parsed were acquired)

## Evaluation signals

- Every parsed MS2 event retains both precursor m/z and retention time fields required for bounding-box matching
- Each MS2 event is attributed to any picked-peak bounding box (RT within the box's RT limits, defined by the first and last MS1 scans of the peak) it falls into; a peak is covered only if at least one event falls within its bounding box
- WeightedDEW output contains a weight column in [0, 1] for every ranked ion, with w = 0 while t - tf <= t0 = 15 s and a linear rise to 1 at t1 = 120 s
- Computed coverage counts per controller per sample (serum QCA and beer QCB) reproduce the paper's Table 1 values, e.g. WeightedDEW as best performer and the TopN gap to optimal (656 vs 1542 serum; 1046 vs 2955 beer)

## Limitations

- Coverage depends on the MZmine 2 peak-picking parameters (Supporting Information Table SI-1); different picking settings change the bounding boxes and hence the coverage counts
- Coverage of a single injection is bounded by the theoretically optimal performance, which accounts for the uneven elution distribution of the ions; parsed-event matching cannot exceed this upper bound
- DDA typically only fragments a small subset of the ions present, so parsed MS2 event lists are sparse relative to the ions detected in the MS1 survey scans

## Evidence

- [methods] MS2 events matched to MZmine bounding boxes: [evidence span withheld — anchor status: not_found]
- [methods] Extraction of MS2 events from six mzML injections: [evidence span withheld — anchor status: not_found]
- [intro] ViMMS in silico validation of parsed DDA strategies: "new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer"
- [abstract] DDA fragmentation subset limitation: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
