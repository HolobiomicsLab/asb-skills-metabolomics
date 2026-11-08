---
name: lc-msms-peak-picking-mzmine
description: Use when raw full-scan LC-MS/MS mzML files (e.g. serum QCA or beer QCB
  extract injections) must be converted into picked-peak RT/mz bounding boxes so that
  DDA controller coverage of picked peaks can be tabulated.
when_to_use_negative:
- Input is already a picked feature/peak table — re-running mzMine 2 peak picking
  would be redundant
- Task is in silico prototyping of DDA controllers in ViMMS rather than evaluating
  coverage on real instrument mzML injections
- Data are DIA or otherwise lack precursor m/z / retention-time MS2 events, since
  coverage is defined by matching fragmentation events to picked peak bounding boxes
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and optimize the
    DDA strategies whose resulting injections are peak-picked and coverage-scored
- name: IAPI
  role: Instrument Application Programming Interface used to run the developed DDA
    controllers on real samples, producing the mzML injections evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:e84adef66e5943bcf3756281a2c5ca0b5a7d5589cb7f261e448956ece429c6a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - ViMMS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-msms-peak-picking-mzmine

## Summary

Peak-pick full-scan LC-MS/MS mzML files with mzMine 2 (Supporting Information Table SI-1 parameters) and export peaks as RT/mz bounding boxes, which are then matched against MS2 fragmentation events to quantify the coverage of DDA controllers such as TopN, SmartROI and WeightedDEW.

## When to use

Use when raw full-scan LC-MS/MS mzML files (e.g. serum QCA or beer QCB extract injections) must be converted into picked-peak RT/mz bounding boxes so that DDA controller coverage of picked peaks can be tabulated.

## When NOT to use

- Input is already a picked feature/peak table — re-running mzMine 2 peak picking would be redundant
- Task is in silico prototyping of DDA controllers in ViMMS rather than evaluating coverage on real instrument mzML injections
- Data are DIA or otherwise lack precursor m/z / retention-time MS2 events, since coverage is defined by matching fragmentation events to picked peak bounding boxes

## Inputs

- Full-scan mzML files for the serum (QCA) and beer (QCB) extracts
- mzML files from the six instrument injections (full-scan, TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW t0 = 15 s / t1 = 120 s, optimal reference)
- mzMine 2 peak-picking parameter set from Supporting Information Table SI-1

## Outputs

- Picked peak list exported as RT/mz bounding boxes
- Per-controller, per-sample coverage counts (number of picked peaks with at least one matched MS2 fragmentation event)
- Coverage table comparable to the paper's Table 1, including gaps to the optimal upper bound

## How to apply

Load the full-scan mzML files for the serum (QCA) and beer (QCB) extracts and pick peaks using mzMine 2 with the parameters provided in the paper's Supporting Information Table SI-1, exporting each peak as an RT/mz bounding box. Load the mzML files from the instrument injections being evaluated (full-scan, TopN, SmartROI shifts, WeightedDEW with t0 = 15 s and t1 = 120 s, optimal reference) and extract MS2 fragmentation events with their precursor m/z and retention times. Match each MS2 event to the picked peak bounding boxes, defining coverage as the number of picked peaks containing at least one fragmentation event. Tabulate coverage counts per controller per sample alongside the optimal upper-bound values (e.g. TopN reaches 656 of 1542 on serum and 1046 of 2955 on beer), which is the correct framework for judging whether a controller like WeightedDEW is the best-performing controller. Rationale: the DDA controller itself only fragments a small subset of ions, so peak picking on the full-scan file provides the ground-truth ion population against which controller coverage is measured.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and optimize the DDA strategies whose resulting injections are peak-picked and coverage-scored)
- **IAPI** (Instrument Application Programming Interface used to run the developed DDA controllers on real samples, producing the mzML injections evaluated)

## Evaluation signals

- Computed WeightedDEW coverage values reproduce the Table 1 values, with WeightedDEW the best performing controller in both serum and beer iterations
- The reported gap to the optimal is consistent with reconstructed counts (656 vs 1542 serum; 1046 vs 2955 beer for TopN)
- Every MS2 fragmentation event is matched to exactly the picked peak whose RT/mz bounding box contains its precursor m/z and retention time
- Coverage is defined identically across all five controllers (number of picked peaks fragmented), so counts are directly comparable per sample

## Limitations

- Coverage depends on the mzMine 2 parameters in SI Table SI-1; different peak-picking settings change the ground-truth peak set and hence coverage counts
- DDA typically only fragments a small subset of the ions present, so coverage of picked peaks is bounded by the optimal reference injection rather than by the full peak list
- Coverage quantification applies to injections where MS2 fragmentation events carry precursor m/z and retention times, i.e. DDA-style acquisitions

## Evidence

- [methods] Load the full-scan mzML files for the serum (QCA) and beer (QCB) extracts and pick peaks using mzMine 2 with the parameters provided in the paper's Supporting Information Table SI-1, exporting peaks as RT/mz bounding boxes.: [evidence span withheld — anchor status: not_found]
- [methods] Match each MS2 fragmentation event to the picked peak bounding boxes to determine which picked peaks contain at least one fragmentation event, defining coverage as the number of picked peaks fragmented.: [evidence span withheld — anchor status: not_found]
- [results] WeightedDEW is the best performing controller in both iterations of the serum and beer extracts, i.e., it attains the highest coverage of picked peaks among the five controllers.: [evidence span withheld — anchor status: not_found]
- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
