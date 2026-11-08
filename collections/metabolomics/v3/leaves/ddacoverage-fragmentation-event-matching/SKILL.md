---
name: ddacoverage-fragmentation-event-matching
description: Use when mzML files from multiple DDA controller injections (e.g., TopN,
  SmartROI, WeightedDEW with t0 = 15 s and t1 = 120 s) must be scored by how many
  picked peaks were fragmented, to rank controllers on coverage.
when_to_use_negative:
- You want to compare controllers on MS2 spectral quality or identification rate rather
  than the proportion of picked peaks fragmented (coverage)
- No full-scan mzML peak-picking reference exists — coverage is defined against picked
  peaks, so without mzMine 2 bounding boxes the matching cannot be performed
- Evaluating simulated rather than real injections; in silico controller evaluation
  is handled separately through ViMMS
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and evaluate DDA
    controller strategies in silico before the coverage matching is run on real injections
- name: IAPI
  role: Instrument Application Programming Interface used to implement the controller
    strategies and acquire the real-sample injections whose MS2 events are matched
    for coverage
- name: mzMine 2
  role: Peak picking of the full-scan mzML files to produce the RT/mz bounding boxes
    that define which peaks count as covered
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:527aba91a14c380a2a3614e59fbc8492ebb7b09f96be48cde06c4fcd3f469ea4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - ViMMS
  - mzMine 2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ddacoverage-fragmentation-event-matching

## Summary

Computes DDA coverage — the number of mzMine-picked peaks that received at least one MS2 fragmentation event — by matching MS2 precursor m/z and retention times against RT/mz bounding boxes, enabling controller comparisons (e.g., TopN vs SmartROI vs WeightedDEW) against the theoretically optimal upper bound.

## When to use

Use when mzML files from multiple DDA controller injections (e.g., TopN, SmartROI, WeightedDEW with t0 = 15 s and t1 = 120 s) must be scored by how many picked peaks were fragmented, to rank controllers on coverage.

## When NOT to use

- You want to compare controllers on MS2 spectral quality or identification rate rather than the proportion of picked peaks fragmented (coverage)
- No full-scan mzML peak-picking reference exists — coverage is defined against picked peaks, so without mzMine 2 bounding boxes the matching cannot be performed
- Evaluating simulated rather than real injections; in silico controller evaluation is handled separately through ViMMS

## Inputs

- Full-scan mzML files of the samples (e.g., serum QCA and beer QCB extracts) for peak picking
- mzMine 2 peak-picking output exported as RT/mz bounding boxes per picked peak (parameters from SI Table SI-1)
- mzML files from each controller injection (TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW t0 = 15 s / t1 = 120 s, optimal reference) containing MS2 fragmentation events with precursor m/z and retention times

## Outputs

- Per-controller, per-sample coverage counts: number of picked peaks containing at least one MS2 fragmentation event
- Mapping of each MS2 fragmentation event to the picked-peak bounding box(es) it falls within
- Comparison table of observed coverage versus the optimal upper-bound coverage per sample

## How to apply

First, pick peaks from the full-scan mzML files (e.g., serum QCA and beer QCB extracts) in mzMine 2 using the SI Table SI-1 parameters and export each peak as an RT/mz bounding box. Next, parse the mzML files from the instrument injections (full-scan, TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW with t0 = 15 s and t1 = 120 s, and the optimal reference) and extract every MS2 fragmentation event with its precursor m/z and retention time. Then match each MS2 event to the picked-peak bounding boxes: a picked peak counts as covered if it contains at least one fragmentation event. Tabulate coverage counts per controller per sample alongside the optimal upper-bound values, and verify that the reconstructed counts reproduce the reported table (e.g., WeightedDEW attains the highest coverage among the five controllers in both serum and beer iterations, while TopN's gap to optimal is 656 vs 1542 for serum and 1046 vs 2955 for beer).

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and evaluate DDA controller strategies in silico before the coverage matching is run on real injections)
- **IAPI** (Instrument Application Programming Interface used to implement the controller strategies and acquire the real-sample injections whose MS2 events are matched for coverage)
- **mzMine 2** (Peak picking of the full-scan mzML files to produce the RT/mz bounding boxes that define which peaks count as covered)

## Evaluation signals

- Computed coverage counts reproduce the published Table 1 values for each controller on serum and beer
- WeightedDEW (t0 = 15 s, t1 = 120 s) is verified to have the highest coverage of picked peaks among the five controllers in both serum and beer iterations
- Reconstructed TopN-to-optimal gap is consistent with the reported values (656 vs 1542 serum; 1046 vs 2955 beer)
- Every MS2 fragmentation event is either matched to a picked-peak bounding box or explicitly counted as unmatched — no events silently dropped
- Coverage of any controller does not exceed the optimal upper bound computed for that injection

## Limitations

- Coverage depends on the mzMine 2 peak-picking parameters (SI Table SI-1); different picking settings change the denominator and thus the coverage counts
- DDA only fragments a small subset of ions present, so coverage should be interpreted against the theoretically optimal upper bound, which accounts for the uneven elution distribution of ions
- Controller parameters are sample/strategy-specific (e.g., WeightedDEW t0 = 15 s and t1 = 120 s); coverage results do not transfer automatically to other parameterizations or sample matrices

## Evidence

- [abstract] DDA typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] Coverage improvement objective: "Here, we address the problem of improving DDA coverage for a single injection"
- [intro] Optimal upper bound rationale: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
- [abstract] In silico prototyping framework: "an in silico framework for fast and cost-efficient development of new DDA strategies"
