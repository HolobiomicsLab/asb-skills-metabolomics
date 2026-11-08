---
name: peak-area-integration
description: Use when processing AEMS MRM raw data (XIC points per mass transition
  and ejection/well) to produce a baseline-corrected area table, e.g. mrm_areas.csv,
  for shot-to-peak quantification.
when_to_use_negative:
- Input is already a peak-area table (mrm_areas.csv) rather than raw AEMS XIC data
  — the areas have already been computed.
- Data are conventional LC-MS chromatograms with retained chromatographic separation
  rather than AEMS ejection windows.
- The acquisition is full-scan AEMS (precursor ion or neutral loss mode) rather than
  targeted MRM transitions.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: R package that automates peak alignment, baseline correction, and area quantification
    for AEMS MRM acquisitions
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: underlying R framework extended by rtmsEcho for raw-data access and XIC handling
- name: R
  role: implementation language for the baseline estimation and windowed area summation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:eb0cea750073d85c93b2fdddf9f47d204775fea584404f755b60bf5c4b430a8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-area-integration

## Summary

Computes per-ejection, per-transition MRM peak areas from acoustic ejection mass spectrometry (AEMS) raw data by baseline-correcting extracted ion chromatograms (XICs) within each detection window and summing intensity, reproducing rtmsEcho's automated area-quantification behavior.

## When to use

Use when processing AEMS MRM raw data (XIC points per mass transition and ejection/well) to produce a baseline-corrected area table, e.g. mrm_areas.csv, for shot-to-peak quantification.

## When NOT to use

- Input is already a peak-area table (mrm_areas.csv) rather than raw AEMS XIC data — the areas have already been computed.
- Data are conventional LC-MS chromatograms with retained chromatographic separation rather than AEMS ejection windows.
- The acquisition is full-scan AEMS (precursor ion or neutral loss mode) rather than targeted MRM transitions.

## Inputs

- AEMS MRM raw data (discrete XIC points per mass transition and ejection)
- detection-window table mapping each well/ejection to its mass transition and time window

## Outputs

- mrm_areas.csv — table with columns transition, ejection/well, area A(m,k)

## How to apply

Load the AEMS MRM raw data and the detection-window table that maps each well/ejection to its mass transition and time window using the rtms/rtmsEcho R framework. For each transition m and ejection k, extract the discrete XIC points falling inside that ejection's detection window. Estimate the per-transition baseline for each ejection by averaging that transition's intensity over two one-second windows, one before the first ejection and one after the last ejection of the run, scaled by 2/3; subtract this estimate from the in-window intensities. Sum the baseline-corrected intensity over each ejection's time window to yield the area A(m,k) for every transition-ejection pair. Assemble the A(m,k) values into a (transition, ejection/well, area) table and save as mrm_areas.csv. Rationale: rtmsEcho automates peak alignment, baseline correction, and area quantification for AEMS, where chromatographic separation is absent and hundreds to thousands of measurements per hour must be processed in seconds (full 384-well plate, <5 transitions, <2 s).

## Related tools

- **rtmsEcho** (R package that automates peak alignment, baseline correction, and area quantification for AEMS MRM acquisitions) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (underlying R framework extended by rtmsEcho for raw-data access and XIC handling)
- **R** (implementation language for the baseline estimation and windowed area summation)

## Examples

```
Rscript rtmsEcho_mrm_pipeline.R --raw aems_mrm.raw --window-table detection_windows.tsv --output mrm_areas.csv
```

## Evaluation signals

- Every transition-ejection pair listed in the detection-window table has a nonmissing A(m,k) entry in mrm_areas.csv.
- Areas reproduce rtmsEcho's baseline-correction and area-quantification behavior (peak alignment, baseline correction, area quantification).
- Per-transition baseline is derived from two one-second windows (pre-first and post-last ejection) scaled by 2/3, not from arbitrary pre-window means.
- Baseline-corrected intensities within each detection window sum to the reported area; uncorrected sums differ.

## Limitations

- Baseline estimation relies on pre-first/post-last ejection windows; if those windows contain signal (e.g. carryover), the baseline estimate and areas are biased.
- Areas are computed from discrete XIC points within each ejection window; no chromatographic peak-shape fitting or smoothing is described.
- Only MRM transitions are covered here; full-scan PI/NL modes require separate processing.

## Evidence

- [methods] baseline estimation from pre/post ejection windows: [evidence span withheld — anchor status: not_found]
- [discussion] automated quantification steps: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [intro] throughput rationale for automation: "This approach enables the precise measurement of targeted MRM transitions or acquisition of full-scan MS data from each well of 384-well or 1536-well microplates, supporting hundreds to thousands of"
