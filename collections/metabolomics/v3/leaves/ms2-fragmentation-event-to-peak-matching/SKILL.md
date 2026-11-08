---
name: ms2-fragmentation-event-to-peak-matching
description: Use when raw mzML injections from a TopN DDA controller and full-scan
  mzML of the same serum (QCA) or beer (QCB) extracts are available, and you must
  measure what fraction of picked peaks received MS2 fragmentation events.
when_to_use_negative:
- The full-scan peaks have not yet been picked — MZmine2 peak picking with SI Table
  SI-1 parameters must precede the bounding-box export.
- The acquisition is DIA rather than DDA, so there are no discrete MS2 fragmentation
  events tied to MS1-survey-selected ions to match against peak bounding boxes.
- Input is already a coverage/feature table rather than mzML injections and picked-peak
  bounding boxes.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MZmine 2
  role: Peak-picks the full-scan QCA/QCB mzML files (SI Table SI-1 parameters) and
    exports picked peaks as RT/m/z bounding boxes to match MS2 events against
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and simulate DDA
    controller strategies whose coverage is measured with this matching procedure
- name: XCMS
  role: Alternative peak-detection tool for generating chromatographic peak bounding
    boxes from full-scan mzML
- name: Peakonly
  role: Alternative peak-detection tool for generating chromatographic peak bounding
    boxes from full-scan mzML
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:7e71c38bacd47a4a875b6d23e1c8782f209d9caff13a140884575ebd4fca6aa2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - Peakonly
  - ViMMS
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms2-fragmentation-event-to-peak-matching

## Summary

Quantifies DDA coverage by matching MS2 fragmentation events from controller-injected mzML files against MS1 peak bounding boxes picked with MZmine2, yielding the proportion of picked peaks fragmented (e.g. 656/1542 serum, 1046/2955 beer) and fragmentation efficiency.

## When to use

Use when raw mzML injections from a TopN DDA controller and full-scan mzML of the same serum (QCA) or beer (QCB) extracts are available, and you must measure what fraction of picked peaks received MS2 fragmentation events.

## When NOT to use

- The full-scan peaks have not yet been picked — MZmine2 peak picking with SI Table SI-1 parameters must precede the bounding-box export.
- The acquisition is DIA rather than DDA, so there are no discrete MS2 fragmentation events tied to MS1-survey-selected ions to match against peak bounding boxes.
- Input is already a coverage/feature table rather than mzML injections and picked-peak bounding boxes.

## Inputs

- Full-scan mzML files for serum (QCA) and beer (QCB) extracts, peak-picked in MZmine2 with SI Table SI-1 parameters
- Picked-peak bounding boxes defined by min/max RT (first and last MS1 scans of the chromatographic peak) and min/max m/z
- mzML files from the TopN DDA controller injections, containing MS1 and MS2 scan records

## Outputs

- Per-sample TopN coverage (count of picked peaks with ≥1 fragmentation event, e.g. 656 serum, 1046 beer)
- Efficiency values (picked peaks fragmented / number of MS2 scans)
- results_table.tsv with per-sample coverage, efficiency, and MS1/MS2 scan counts (Table 2 comparison)

## How to apply

First pick peaks from the full-scan mzML files of the serum (QCA) and beer (QCB) extracts in MZmine2 using the SI Table SI-1 parameters, then export each peak as a bounding box whose min/max RT span is defined by the first and last MS1 scans of the chromatographic peak and whose m/z bounds bracket the peak. Next, parse the mzML files from the TopN controller injections and extract all MS2 fragmentation events. Match each MS2 event to the peak bounding boxes it falls into (if any), using both the RT range and m/z range of the box. Compute coverage as the number of picked peaks containing at least one fragmentation event and efficiency as the ratio of picked peaks fragmented to the number of MS2 scans. Rationale: DDA 'typically only fragments a small subset of the ions present', so comparing observed coverage against the theoretically optimal coverage (1542 serum, 2955 beer peaks) quantifies the room for improvement over the standard controller.

## Related tools

- **MZmine 2** (Peak-picks the full-scan QCA/QCB mzML files (SI Table SI-1 parameters) and exports picked peaks as RT/m/z bounding boxes to match MS2 events against)
- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and simulate DDA controller strategies whose coverage is measured with this matching procedure)
- **XCMS** (Alternative peak-detection tool for generating chromatographic peak bounding boxes from full-scan mzML)
- **Peakonly** (Alternative peak-detection tool for generating chromatographic peak bounding boxes from full-scan mzML)

## Evaluation signals

- Observed TopN coverage must match Table 1: 656 fragmented peaks on the serum extract vs 1542 optimal, and 1046 vs 2955 on the beer extract.
- Per-sample MS1 and MS2 scan counts written to results_table.tsv must match the values in Table 2.
- Coverage can never exceed the theoretically optimal peak count for that injection, which accounts for the uneven elution distribution of the ions.
- Every MS2 event either falls within exactly one or more peak bounding boxes (RT and m/z ranges) or is correctly discarded as unmatched.
- Efficiency must equal picked peaks fragmented divided by the number of MS2 scans for each sample.

## Limitations

- Coverage depends on the MZmine2 peak-picking parameters (SI Table SI-1); different pickers (XCMS, Peakonly) or thresholds change the bounding-box set and thus the measured coverage.
- Bounding boxes derived from first/last MS1 scans of a peak may mis-assign MS2 events near overlapping or co-eluting peaks.
- The theoretical optima (1542 serum, 2955 beer) are injection-specific bounds; observed coverage below them reflects both controller behaviour and the unequal elution distribution of ions.

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] Theoretical optimum accounts for uneven elution: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
- [methods] Workflow step: match MS2 events to bounding boxes: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: compute coverage and efficiency: [evidence span withheld — anchor status: not_found]
