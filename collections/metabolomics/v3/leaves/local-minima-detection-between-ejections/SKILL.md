---
name: local-minima-detection-between-ejections
description: Use when you have an AEMS run with a per-ejection time table and a TIC
  trace, and need to assign each ejection a bounded detection window (min_time, peak_time,
  max_time) before peak integration or quantification.
when_to_use_negative:
- The acquisition is conventional LC-MS with continuous chromatographic separation
  rather than discrete acoustic ejections from a 384- or 1536-well plate
- No upstream ejection-time table exists — the per-ejection timing must first be inferred
  before windows can be bracketed
- You only need downstream steps (peak integration, area quantification, spectral
  analysis) and windows are already available
tools:
- name: rtmsEcho
  role: Implements the TIC smoothing, local-minima search between adjacent ejection
    times, background-noise thresholding, and per-ejection window output
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework extended by rtmsEcho; used to load raw AEMS data and
    extract the TIC trace
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: R
  role: Environment in which the rtmsEcho package and this windowing step run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:0dd1db213d3cbc7a3e13ed695d1cedceadbc0b9e4a44c75709976ea6890bfa04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# local-minima-detection-between-ejections

## Summary

A rtmsEcho windowing step that defines the detection window of each acoustic ejection in an AEMS run by lightly smoothing the TIC intensity trace and locating local minima between adjacent ejection times, bounded either by the background-noise threshold or by the smoothed-trace minimum.

## When to use

Use when you have an AEMS run with a per-ejection time table and a TIC trace, and need to assign each ejection a bounded detection window (min_time, peak_time, max_time) before peak integration or quantification.

## When NOT to use

- The acquisition is conventional LC-MS with continuous chromatographic separation rather than discrete acoustic ejections from a 384- or 1536-well plate
- No upstream ejection-time table exists — the per-ejection timing must first be inferred before windows can be bracketed
- You only need downstream steps (peak integration, area quantification, spectral analysis) and windows are already available

## Inputs

- Ejection-time table produced by the upstream timing-inference step
- TIC (total ion current) intensity trace extracted from raw AEMS data
- Estimated background-noise level of the TIC trace

## Outputs

- Per-ejection detection-window records with peak_time, min_time, and max_time
- ejection_windows.csv (assembled window table)

## How to apply

Load the upstream ejection-time table and the TIC trace extracted from the raw AEMS data using rtmsEcho/rtms in R. Apply a slight smoothing filter to the TIC so shot-to-shot fluctuations do not create spurious minima. For each pair of adjacent ejection times, scan the smoothed trace in the intervening interval and locate the local minima flanking the expected peak. Estimate the background-noise level and use it as a threshold: window bounds are set where total intensity falls to the noise threshold, or — if the signal never drops below that threshold between adjacent ejections — at the local minimum of the smoothed total intensity; windows whose signal never exceeds the noise threshold are rejected. For each accepted ejection, record peak_time (local signal maximum within the window) and min_time/max_time (the flanking local minima defining the bounds), and write the per-ejection records as ejection_windows.csv.

## Related tools

- **rtmsEcho** (Implements the TIC smoothing, local-minima search between adjacent ejection times, background-noise thresholding, and per-ejection window output) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework extended by rtmsEcho; used to load raw AEMS data and extract the TIC trace) — https://github.com/nathanieltwarog/rtmsEcho
- **R** (Environment in which the rtmsEcho package and this windowing step run)

## Evaluation signals

- Every accepted ejection yields exactly one peak_time lying strictly between its min_time and max_time
- Windows whose TIC signal does not rise above the background-noise threshold are excluded from the output
- The number of window records matches the number of accepted ejections in the upstream ejection-time table
- Adjacent windows do not overlap: each ejection's max_time coincides with (or abuts) the next ejection's min_time at the shared local minimum or noise-threshold crossing

## Limitations

- The smoothing must be slight — over-smoothing can merge features across adjacent ejections, under-smoothing lets shot-to-shot fluctuations create spurious minima
- When the TIC never drops below the background-noise threshold between adjacent ejections, the window boundary defaults to the smoothed-intensity local minimum, which is a weaker boundary than a true return to noise
- No benchmark datasets or accessioned data resources are named to validate windowing behavior; no software version number is stated for rtmsEcho

## Evidence

- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating"
- [abstract] shot-to-peak association: "while automating shot-to-peak association and spectral analysis"
- [abstract] peak detection: "critical tasks such as peak detection, integration, and multimodal data analysis"
- [discussion] automated processing steps: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
