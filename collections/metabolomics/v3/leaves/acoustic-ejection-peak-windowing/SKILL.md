---
name: acoustic-ejection-peak-windowing
description: Use when you have an AEMS run with a table of inferred ejection times
  and an extracted TIC trace, and you must define each ejection's detection window
  bounded by flanking local minima.
when_to_use_negative:
- Input is raw vendor AEMS data with no upstream ejection-time table — this skill
  assumes ejection times have already been inferred
- The goal is peak-area quantification or baseline correction rather than determining
  per-ejection window boundaries
- Data are conventional LC-MS chromatograms rather than acoustic ejection mass spectrometry
  (AEMS) runs
tools:
- name: rtmsEcho
  role: R package that extends rtms to load AEMS data, extract the TIC trace, and
    automate shot-to-peak association and per-ejection window determination
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework extended by rtmsEcho, providing the mass-spectrometry
    data handling used to extract and smooth the TIC signal
  repo: github.com/nathanieltwarog/rtmsEcho
- name: R
  role: Language in which the smoothing, local-minimum detection, and background-noise
    thresholding are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:e0f1a2bdd330eefbe5da42bfd3972341062a4197fae0702e9e3d1d6a498f3bfe
schema_version: 0.2.0
metadata:
  license_tier: open
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
n_claims: 7
---

# acoustic-ejection-peak-windowing

## Summary

Delimits the per-ejection detection window in an AEMS run by smoothing the TIC intensity trace and locating local minima between adjacent ejection times, then recording peak_time and min_time/max_time per ejection (e.g. as ejection_windows.csv).

## When to use

Use when you have an AEMS run with a table of inferred ejection times and an extracted TIC trace, and you must define each ejection's detection window bounded by flanking local minima.

## When NOT to use

- Input is raw vendor AEMS data with no upstream ejection-time table — this skill assumes ejection times have already been inferred
- The goal is peak-area quantification or baseline correction rather than determining per-ejection window boundaries
- Data are conventional LC-MS chromatograms rather than acoustic ejection mass spectrometry (AEMS) runs

## Inputs

- Ejection-time table produced by the upstream timing-inference step for the AEMS run
- TIC intensity trace extracted from the raw AEMS data via rtmsEcho/rtms in R
- Estimated background-noise level of the TIC trace (or trace data sufficient to estimate it)

## Outputs

- Per-ejection detection-window records with peak_time, min_time, and max_time
- ejection_windows.csv table of accepted per-ejection windows

## How to apply

Load the ejection-time table from the upstream timing-inference step and the TIC trace extracted from the raw AEMS data using rtmsEcho/rtms in R. Apply a slight local smoothing filter to the TIC signal so shot-to-shot fluctuations do not create spurious minima. For each pair of adjacent ejection times, scan the smoothed trace within the intervening interval and identify the local minima bracketing the expected peak; estimate the background-noise level of the trace and use it as a threshold, setting window boundaries either where total intensity falls to that threshold or, if it never drops below it between adjacent ejections, at the local minimum in smoothed total intensity. For each accepted ejection record peak_time (local signal maximum within the window) and min_time/max_time (flanking local minima defining the window bounds), then assemble the per-ejection window records into ejection_windows.csv.

## Related tools

- **rtmsEcho** (R package that extends rtms to load AEMS data, extract the TIC trace, and automate shot-to-peak association and per-ejection window determination) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework extended by rtmsEcho, providing the mass-spectrometry data handling used to extract and smooth the TIC signal) — github.com/nathanieltwarog/rtmsEcho
- **R** (Language in which the smoothing, local-minimum detection, and background-noise thresholding are implemented)

## Evaluation signals

- Each accepted ejection must yield exactly one peak_time lying strictly between its min_time and max_time
- Windows whose signal does not exceed the background-noise threshold must be excluded from the output
- Every pair of adjacent ejections contributes at most one window; the count of accepted windows must not exceed the number of ejections
- min_time and max_time for each ejection must coincide with local minima of the smoothed (not raw) TIC trace
- The assembled ejection_windows.csv must contain one row per accepted ejection with all three time columns populated

## Limitations

- When total intensity never drops below the background-noise threshold between adjacent ejections, the window boundary falls back to the local minimum in smoothed intensity, which may be less precise than a threshold crossing
- The smoothing step is 'slight' by design; overly aggressive smoothing would shift the local minima that define window boundaries
- No software version, benchmark dataset, or accessioned resource is provided in the extracted article context to validate windowing behavior

## Evidence

- [abstract] while automating shot-to-peak association and spectral analysis: "while automating shot-to-peak association and spectral analysis"
- [other] rtmsEcho determines the time range during which each ejection is detected: [evidence span withheld — anchor status: not_found]
- [other] each ejection must yield exactly one peak_time lying between its min_time and max_time: [evidence span withheld — anchor status: not_found]
