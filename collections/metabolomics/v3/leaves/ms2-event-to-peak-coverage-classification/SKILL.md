---
name: ms2-event-to-peak-coverage-classification
description: Use when you have an mzML file containing MS2 fragmentation events plus
  picked-peak bounding boxes (m/z and RT min/max, e.g. exported from MZmine2) and
  need per-peak coverage flags to quantify fragmentation-controller coverage.
when_to_use_negative:
- The input is already a coverage summary or feature table — this skill expects raw
  fragmentation events and bounding boxes, not pre-aggregated results.
- Peak definitions are apex points or single RT values rather than RT/m/z bounding
  boxes spanning the first and last MS1 scans of the peak.
- The acquisition is DIA or otherwise lacks discrete MS2 fragmentation events tied
  to MS1 survey-scan intensities, so per-event attribution to picked peaks is not
  meaningful.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Source of the picked-peak bounding boxes (m/z and RT min/max) that MS2 events
    are tested against for coverage attribution
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to generate simulated mzML runs
    for in silico DDA strategy development whose fragmentation coverage can be scored
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:4e3a612f0dff227589bacfe3fcc8bb83c1f8bd0adeeccfaaf70f36539b181988
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# ms2-event-to-peak-coverage-classification

## Summary

Scores how well a data-dependent acquisition (DDA) run covered a picked-peak list by attributing each MS2 fragmentation event from an mzML file to MZmine2 peak bounding boxes (m/z and RT min/max) and flagging each picked peak as covered if at least one MS2 event falls inside it.

## When to use

Use when you have an mzML file containing MS2 fragmentation events plus picked-peak bounding boxes (m/z and RT min/max, e.g. exported from MZmine2) and need per-peak coverage flags to quantify fragmentation-controller coverage.

## When NOT to use

- The input is already a coverage summary or feature table — this skill expects raw fragmentation events and bounding boxes, not pre-aggregated results.
- Peak definitions are apex points or single RT values rather than RT/m/z bounding boxes spanning the first and last MS1 scans of the peak.
- The acquisition is DIA or otherwise lacks discrete MS2 fragmentation events tied to MS1 survey-scan intensities, so per-event attribution to picked peaks is not meaningful.

## Inputs

- mzML file containing MS2 fragmentation events with retention times (and precursor m/z where available)
- Picked-peak bounding boxes (m/z and RT min/max, RT limits defined by first and last MS1 scans of each peak), e.g. exported from MZmine 2

## Outputs

- coverage_flags.csv — per-peak table with bounding-box coordinates (m/z and RT min/max) and a boolean coverage flag
- Coverage count/fraction — number (or proportion) of picked peaks containing at least one MS2 fragmentation event

## How to apply

Read the mzML file and extract all MS2 fragmentation events with their retention times (and precursor m/z if needed). Load the picked-peak bounding boxes for the corresponding full-scan run, where each box is defined by m/z and RT min/max and the RT range is set by the first and last MS1 scans comprising the peak. For each MS2 event, test whether it falls inside a peak bounding box — the attribution rule is containment: an event is attributed to any bounding box it falls into (RT within the box's RT limits). Flag each picked peak as covered if at least one MS2 event falls within its bounding box, and write the per-peak coverage flag table as coverage_flags.csv with columns for bounding-box coordinates and a boolean coverage flag. This metric is motivated by the DDA setting where only a small subset of ions present is fragmented, so coverage measures the proportion of picked peaks that received at least one fragmentation event in a single injection.

## Related tools

- **MZmine 2** (Source of the picked-peak bounding boxes (m/z and RT min/max) that MS2 events are tested against for coverage attribution)
- **ViMMS** (Virtual metabolomics mass spectrometer used to generate simulated mzML runs for in silico DDA strategy development whose fragmentation coverage can be scored)

## Evaluation signals

- Every MS2 event is attributed to every bounding box it falls into (containment test on RT and m/z limits), not just the closest peak.
- A picked peak is marked covered only when at least one MS2 event lies within its bounding box; peaks with zero contained events must be flagged False.
- coverage_flags.csv contains columns for the bounding-box coordinates (m/z and RT min/max) and a boolean coverage flag for each picked peak.
- The reported coverage equals the count of picked peaks containing at least one fragmentation event, consistent with the finding's definition of coverage.

## Limitations

- Coverage depends on how peaks were picked — the RT range is set by the first and last MS1 scans comprising the peak, so changes in peak-picking parameters alter the bounding boxes and hence the coverage score.
- An MS2 event is attributed to any bounding box it falls into, so overlapping boxes (e.g. co-eluting or isobaric peaks) can each be credited by the same event.
- Coverage here reflects DDA behaviour, where typically only a small subset of the ions present is fragmented, so absolute coverage values are bounded by instrument cycle times and prioritization strategy rather than peak-picking quality alone.

## Evidence

- [other] DDA event attribution to bounding boxes: [evidence span withheld — anchor status: not_found]
- [other] Coverage definition: [evidence span withheld — anchor status: not_found]
- [abstract] DDA subset-fragmentation motivation: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] Coverage-improvement objective: "Here, we address the problem of improving DDA coverage for a single injection"
