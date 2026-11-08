---
name: background-noise-threshold-estimation
description: Use when a TIC trace from acoustic ejection mass spectrometry (AEMS)
  data must be split into per-ejection detection windows bounded by a background-noise
  threshold or smoothed local minima between adjacent ejection times.
when_to_use_negative:
- Data are conventional LC-MS chromatography rather than AEMS acquisitions — this
  windowing is specific to ejection-based runs where shot-to-peak association is required
- No upstream ejection-time table exists yet; the timing-inference step must be run
  first before windows can be bounded
- The goal is concentration quantification or peak-area integration rather than determining
  per-ejection detection-window boundaries
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: Implements TIC smoothing, local-minimum detection, and background-noise thresholding
    to define per-ejection detection windows in AEMS runs
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework extended by rtmsEcho; provides access to and extraction
    of the TIC signal from raw AEMS data
- name: R
  role: Implementation language in which the smoothing, minima detection, and thresholding
    workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:bb451e92a8e51bc5a019384a0850132f517e725328a3e724c422f269283006d3
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
yes_rate: 1.0
n_claims: 6
---

# background-noise-threshold-estimation

## Summary

rtmsEcho's method for bounding the detection window of each acoustic ejection in an AEMS run: slightly smooth the TIC intensity trace, locate local minima between adjacent ejection times, and set window boundaries either where total intensity falls to the background-noise threshold or at the smoothed local minimum.

## When to use

Use when a TIC trace from acoustic ejection mass spectrometry (AEMS) data must be split into per-ejection detection windows bounded by a background-noise threshold or smoothed local minima between adjacent ejection times.

## When NOT to use

- Data are conventional LC-MS chromatography rather than AEMS acquisitions — this windowing is specific to ejection-based runs where shot-to-peak association is required
- No upstream ejection-time table exists yet; the timing-inference step must be run first before windows can be bounded
- The goal is concentration quantification or peak-area integration rather than determining per-ejection detection-window boundaries

## Inputs

- Ejection-time table produced by the upstream timing-inference step
- TIC intensity trace extracted from raw AEMS data (via rtmsEcho/rtms in R)
- Estimated background-noise level of the TIC trace

## Outputs

- Per-ejection window records with peak_time, min_time, and max_time
- ejection_windows.csv table of accepted ejection windows (windows below the background-noise threshold excluded)

## How to apply

Load the ejection-time table from the upstream timing-inference step and the TIC trace extracted from raw AEMS data via rtmsEcho/rtms in R. Apply a slight smoothing filter to the TIC so shot-to-shot fluctuations do not create spurious minima. For each pair of adjacent ejection times, scan the smoothed trace in the intervening interval and identify the local minima bracketing the expected peak; set window boundaries where total intensity falls to the estimated background-noise threshold, or, if it never drops below that threshold between adjacent ejections, at the local minimum in smoothed total intensity. Estimate the background-noise level and reject windows whose signal does not exceed it. For each accepted ejection, record peak_time (local signal maximum within the window) and min_time/max_time (flanking local minima defining the bounds), and assemble the records into ejection_windows.csv.

## Related tools

- **rtmsEcho** (Implements TIC smoothing, local-minimum detection, and background-noise thresholding to define per-ejection detection windows in AEMS runs) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework extended by rtmsEcho; provides access to and extraction of the TIC signal from raw AEMS data)
- **R** (Implementation language in which the smoothing, minima detection, and thresholding workflow is executed)

## Evaluation signals

- Each accepted ejection yields exactly one peak_time lying strictly between its min_time and max_time
- Windows whose total intensity does not exceed the background-noise threshold are excluded from ejection_windows.csv
- Every window boundary falls either at a background-noise threshold crossing or at a local minimum in the smoothed TIC intensity
- One and only one window is produced per accepted ejection time from the upstream timing table
- min_time and max_time are local minima of the smoothed trace flanking the expected peak, not raw noisy minima

## Limitations

- When total intensity never drops below the background-noise threshold between adjacent ejections, the boundary defaults to the local minimum in smoothed intensity, which is a less precise cutoff than a threshold crossing
- The smoothing filter must be slight; shot-to-shot fluctuations in the unsmoothed TIC create spurious minima that corrupt window boundaries
- Ejections whose signal never rises above the background-noise level are rejected entirely, so genuinely weak wells may be missing from the output windows

## Evidence

- [full_text] rtmsEcho determines the time range during which each ejection is detected by performing a slight smoothing of the TIC intensity signal and locating the local minima between adjacent peak times: [evidence span withheld — anchor status: not_found]
- [full_text] Window boundaries at threshold or local minimum: [evidence span withheld — anchor status: not_found]
- [full_text] Validation invariant for accepted windows: [evidence span withheld — anchor status: not_found]
- [abstract] Automated processing gap in existing AEMS tools: "existing tools lack robust automated processing capabilities for critical tasks such as peak detection, integration, and multimodal"
- [abstract] rtmsEcho as an extension of the rtms framework: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
