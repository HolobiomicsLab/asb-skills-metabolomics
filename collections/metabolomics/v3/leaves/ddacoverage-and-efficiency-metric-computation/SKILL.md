---
name: ddacoverage-and-efficiency-metric-computation
description: Use when you have full-scan mzML peak-picking files plus matching DDA
  (TopN) controller mzML runs and need to measure how many picked peaks were actually
  fragmented and how the acquisition compares to the theoretical optimum.
when_to_use_negative:
- You already have a feature/peak table and simply want abundance quantification —
  this skill measures fragmentation coverage, not peak intensity quantification
- Your acquisition is DIA or another non-DDA mode without intensity-based TopN ion
  selection, where matching MS2 events to MS1-picked bounding boxes does not reflect
  the acquisition logic
- You lack a full-scan (MS1-only) peak-picking run corresponding to the TopN injection,
  so no valid peak bounding boxes can be defined
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Peak-picks the full-scan mzML files under SI Table SI-1 parameters and exports
    picked peaks as RT/m/z bounding boxes
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to simulate/validate DDA controller
    injections whose coverage this metric evaluates
- name: XCMS
  role: Alternative peak-detection tool for deriving the picked peaks that define
    the bounding boxes
- name: Peakonly
  role: Alternative peak-detection tool for deriving the picked peaks that define
    the bounding boxes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:9647f9fcbd5c82cd669cc94a5678d4b4ff0a82f0851295c00dfbd24a10d2d342
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ddacoverage-and-efficiency-metric-computation

## Summary

Quantifies how well a DDA acquisition (e.g. standard TopN) covered the ion population of an injection by matching MS2 fragmentation events to MZmine2-picked peak bounding boxes, reporting coverage (proportion of picked peaks fragmented) and efficiency relative to a theoretically optimal value.

## When to use

Use when you have full-scan mzML peak-picking files plus matching DDA (TopN) controller mzML runs and need to measure how many picked peaks were actually fragmented and how the acquisition compares to the theoretical optimum.

## When NOT to use

- You already have a feature/peak table and simply want abundance quantification — this skill measures fragmentation coverage, not peak intensity quantification
- Your acquisition is DIA or another non-DDA mode without intensity-based TopN ion selection, where matching MS2 events to MS1-picked bounding boxes does not reflect the acquisition logic
- You lack a full-scan (MS1-only) peak-picking run corresponding to the TopN injection, so no valid peak bounding boxes can be defined

## Inputs

- Full-scan mzML files for the sample injections (serum QCA and beer QCB extracts)
- TopN controller injection mzML files containing MS2 fragmentation events
- MZmine2 peak-picking parameters (SI Table SI-1)
- Exported peak bounding boxes defined by min/max retention time and min/max m/z

## Outputs

- results_table.tsv with per-sample TopN coverage (number of picked peaks with at least one fragmentation event)
- Per-sample efficiency (fragmented picked peaks divided by number of MS2 scans)
- Counts of MS1 and MS2 scans produced per sample
- Theoretical optimum coverage for comparison (1542 peaks serum, 2955 peaks beer)

## How to apply

First load the full-scan mzML files for the sample (e.g. serum QCA and beer QCB extracts) and pick peaks with MZmine2 using SI Table SI-1 parameters, exporting each picked peak as a bounding box with min/max RT and m/z, where the RT range is defined by the first and last MS1 scans comprising the chromatographic peak. Then read the mzML from the TopN controller injection and extract its MS2 fragmentation events. Match each MS2 event to the peak bounding box(es) it falls inside by RT and m/z; a picked peak counts as fragmented if it contains at least one MS2 event. Compute coverage as the number of picked peaks containing at least one fragmentation event and efficiency as the ratio of picked peaks fragmented to the number of MS2 scans. Because DDA 'typically only fragments a small subset of the ions present', benchmark the observed coverage against the theoretically optimal performance computed for the same injection, 'taking into account the uneven elution distribution of the ions'.

## Related tools

- **MZmine 2** (Peak-picks the full-scan mzML files under SI Table SI-1 parameters and exports picked peaks as RT/m/z bounding boxes)
- **ViMMS** (Virtual metabolomics mass spectrometer used to simulate/validate DDA controller injections whose coverage this metric evaluates)
- **XCMS** (Alternative peak-detection tool for deriving the picked peaks that define the bounding boxes)
- **Peakonly** (Alternative peak-detection tool for deriving the picked peaks that define the bounding boxes)

## Evaluation signals

- Reported TopN coverage matches Table 1 reference values: 656 fragmented peaks for serum (vs 1542 theoretical optimum) and 1046 for beer (vs 2955)
- Per-sample MS1 and MS2 scan counts match Table 2
- Each peak bounding box RT range equals the first and last MS1 scans comprising the chromatographic peak
- Efficiency equals fragmented picked peaks divided by total MS2 scans for that injection
- Observed coverage is strictly below the theoretically optimal coverage computed for the same injection

## Limitations

- Coverage depends on the peak-picking tool and its parameters (MZmine2 SI Table SI-1 parameters; XCMS and Peakonly are alternatives), so results vary with picking thresholds
- The metric evaluates a single injection against a theoretical optimum, not cross-run reproducibility; it applies to DDA-style intensity-selected fragmentation

## Evidence

- [abstract] coverage definition / motivation: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] theoretical optimum rationale: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
- [intro] observed coverage values and goal: "Here, we address the problem of improving DDA coverage for a single injection"
- [results] validation against reported values: [evidence span withheld — anchor status: not_found]
