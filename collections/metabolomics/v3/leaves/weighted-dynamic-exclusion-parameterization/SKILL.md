---
name: weighted-dynamic-exclusion-parameterization
description: Use when tuning or benchmarking DDA controller parameters (e.g., WeightedDEW
  exclusion windows t0/t1) and you need to measure fragmentation coverage of picked
  peaks in mzML injections of complex mixtures against TopN and optimal upper bounds.
when_to_use_negative:
- Input acquisitions are DIA/SWATH rather than DDA MS1-survey-driven fragmentation,
  so dynamic exclusion windows do not apply
- MS2 fragmentation-event-to-peak assignment is already provided (e.g., a precomputed
  coverage table), making the bounding-box matching step redundant
- Peak picking has not been performed and mzMine 2 RT/mz bounding boxes are unavailable,
  so coverage cannot be defined against picked peaks
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype, optimize, and validate
    the WeightedDEW controller parameters in silico before instrument deployment
- name: IAPI
  role: Instrument Application Programming Interface through which the parameterized
    WeightedDEW controller is deployed on real samples (serum and beer extract injections)
- name: mzMine 2
  role: Peak picking of full-scan mzML files into RT/mz bounding boxes that define
    the coverage target set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:658103e303ed1fcdb151fcc909719dd66564cc1766d20d09d38501181b03207d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - ViMMS
  - mzMine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# weighted-dynamic-exclusion-parameterization

## Summary

Parameterize and validate the WeightedDEW data-dependent acquisition (DDA) controller — a weighted dynamic exclusion window strategy (here t0 = 15 s, t1 = 120 s) — and quantify its coverage of mzMine-picked peaks against TopN, SmartROI, and theoretically optimal baselines on complex serum (QCA) and beer (QCB) extracts.

## When to use

Use when tuning or benchmarking DDA controller parameters (e.g., WeightedDEW exclusion windows t0/t1) and you need to measure fragmentation coverage of picked peaks in mzML injections of complex mixtures against TopN and optimal upper bounds.

## When NOT to use

- Input acquisitions are DIA/SWATH rather than DDA MS1-survey-driven fragmentation, so dynamic exclusion windows do not apply
- MS2 fragmentation-event-to-peak assignment is already provided (e.g., a precomputed coverage table), making the bounding-box matching step redundant
- Peak picking has not been performed and mzMine 2 RT/mz bounding boxes are unavailable, so coverage cannot be defined against picked peaks

## Inputs

- Full-scan mzML files for serum (QCA) and beer (QCB) extracts
- mzML files from six instrument injections (full-scan, TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW t0=15 s/t1=120 s, optimal reference)
- mzMine 2 picked peak RT/mz bounding boxes exported with SI Table SI-1 peak-picking parameters
- WeightedDEW controller parameters (dynamic exclusion window settings, here t0 = 15 s, t1 = 120 s)

## Outputs

- Coverage counts per controller per sample (number of picked peaks containing ≥1 MS2 fragmentation event)
- Comparison table (cf. Table 1) of WeightedDEW, TopN, SmartROI, and optimal upper-bound coverage for serum and beer

## How to apply

Load full-scan mzML files for the serum (QCA) and beer (QCB) extracts and pick peaks with mzMine 2 using the parameters in Supporting Information Table SI-1, exporting peaks as RT/mz bounding boxes. Load mzML files from the instrument injections (full-scan, TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW with t0 = 15 s and t1 = 120 s, and the optimal reference) and extract MS2 fragmentation events with precursor m/z and retention times. Match each MS2 event to the picked peak bounding boxes to define coverage as the number of picked peaks containing at least one fragmentation event. Prototype and optimize the controller parameters in silico with ViMMS before deploying WeightedDEW on real hardware through the Instrument Application Programming Interface (IAPI). Tabulate coverage per controller per sample alongside the optimal upper-bound values and verify WeightedDEW attains the highest observed coverage in both serum and beer iterations.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype, optimize, and validate the WeightedDEW controller parameters in silico before instrument deployment)
- **IAPI** (Instrument Application Programming Interface through which the parameterized WeightedDEW controller is deployed on real samples (serum and beer extract injections))
- **mzMine 2** (Peak picking of full-scan mzML files into RT/mz bounding boxes that define the coverage target set)

## Evaluation signals

- WeightedDEW coverage of picked peaks exceeds all other DDA controllers (TopN, SmartROI shifts) in BOTH serum and beer iterations
- Computed WeightedDEW coverage counts reproduce the reported Table 1 values for serum and beer
- The reported gap to the optimal upper bound is consistent with reconstructed counts (e.g., 656 vs 1542 serum; 1046 vs 2955 beer for TopN)
- Only picked peaks containing at least one matched MS2 fragmentation event (precursor m/z and RT within the bounding box) are counted toward coverage
- Peak-picking parameters match SI Table SI-1 so the bounding-box target set is reproducible

## Limitations

- DDA typically fragments only a small subset of ions present, so even the best controller leaves large gaps to the theoretical optimum (TopN: 656 vs 1542 serum; 1046 vs 2955 beer)
- Coverage depends on the mzMine 2 peak-picking parameters from SI Table SI-1; different peak-picking settings change the target bounding-box set and thus the measured coverage
- Validated on only two complex mixtures (serum and beer extracts); generalization to other sample types or chromatographic layouts is not established here

## Evidence

- [other] Coverage definition via matching to picked peaks: [evidence span withheld — anchor status: not_found]
- [other] Optimized WeightedDEW parameters: [evidence span withheld — anchor status: not_found]
- [abstract] In silico prototyping rationale: "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [abstract] Real-instrument deployment via IAPI: "on real samples via an Instrument Application Programming Interface (IAPI)"
- [abstract] Motivation: DDA coverage is a small subset: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] Controller improvement claim: "development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies."
