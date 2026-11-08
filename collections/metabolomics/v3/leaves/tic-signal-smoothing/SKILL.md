---
name: tic-signal-smoothing
description: Use when you have a TIC trace extracted from raw AEMS data plus an upstream
  table of inferred ejection times, and you need to define the time window during
  which each ejection was detected.
when_to_use_negative:
- Input is raw unsmoothed AEMS data without an upstream ejection-time table — this
  skill depends on pre-inferred ejection times to bracket the intervals to scan
- Signal between adjacent ejections never exceeds the estimated background-noise threshold
  — such windows must be rejected, not integrated
edam_operation: http://edamontology.org/operation_0284
tools:
- name: rtmsEcho
  role: Performs the TIC smoothing and local-minima/threshold windowing of AEMS runs
    to determine per-ejection detection windows
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework extended by rtmsEcho, used to load and extract the
    TIC trace from raw AEMS data
- name: R
  role: Language in which the smoothing and windowing pipeline is implemented and
    run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:4caf6d59a45a65335d5e2e77307a28bb60532649420c9220f748639cc60f49d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0284
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tic-signal-smoothing

## Summary

Smooth the total ion current (TIC) intensity trace from acoustic ejection mass spectrometry (AEMS) runs so that per-ejection detection windows can be bounded by background-noise thresholds and local minima rather than by shot-to-shot intensity fluctuations. This is the windowing step rtmsEcho uses before peak integration and shot-to-peak association.

## When to use

Use when you have a TIC trace extracted from raw AEMS data plus an upstream table of inferred ejection times, and you need to define the time window during which each ejection was detected.

## When NOT to use

- Input is raw unsmoothed AEMS data without an upstream ejection-time table — this skill depends on pre-inferred ejection times to bracket the intervals to scan
- Signal between adjacent ejections never exceeds the estimated background-noise threshold — such windows must be rejected, not integrated

## Inputs

- TIC intensity trace extracted from raw AEMS data (time-intensity series)
- Ejection-time table from the upstream timing-inference step (per-shot ejection times)

## Outputs

- Per-ejection window records (peak_time, min_time, max_time) assembled into ejection_windows.csv

## How to apply

Load the ejection-time table produced by the upstream timing-inference step together with the TIC trace extracted from the raw AEMS data using rtmsEcho/rtms in R. Apply a local smoothing filter to the TIC signal so that shot-to-shot fluctuations do not create spurious minima, then, for each pair of adjacent ejection times, scan the smoothed trace within the intervening interval to locate the local minima bracketing the expected peak. Estimate the background-noise level of the trace and use it as a threshold: window boundaries are set where total intensity falls to the background-noise threshold, or, if intensity never drops below that threshold between adjacent ejections, at the local minimum in smoothed total intensity. For each accepted ejection record peak_time (the local signal maximum within the window) and min_time/max_time (the flanking local minima defining the window bounds), and assemble these per-ejection window records into ejection_windows.csv.

## Related tools

- **rtmsEcho** (Performs the TIC smoothing and local-minima/threshold windowing of AEMS runs to determine per-ejection detection windows) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework extended by rtmsEcho, used to load and extract the TIC trace from raw AEMS data)
- **R** (Language in which the smoothing and windowing pipeline is implemented and run)

## Evaluation signals

- Each ejection yields exactly one peak_time that lies between its min_time and max_time
- Windows whose signal does not exceed the estimated background-noise threshold are excluded from the output table
- min_time and max_time correspond to local minima in the smoothed TIC trace bracketing the expected peak for each ejection
- No spurious minima are introduced by shot-to-shot fluctuations (smoothing removed single-shot dips within the inter-ejection interval)

## Limitations

- Windows where the TIC never drops below the background-noise threshold between adjacent ejections must fall back to the smoothed local minimum, so boundary placement is approximate in that regime
- Output depends on the accuracy of the upstream ejection-time table; incorrect inferred ejection times will mis-bracket the scanned intervals
- Over-smoothing could shift or merge adjacent minima, so the local smoothing filter strength must be tuned to the AEMS shot spacing

## Evidence

- [abstract] rtmsEcho enables customizable processing of AEMS acquisitions while automating shot-to-peak association: "while automating shot-to-peak association and spectral analysis"
- [abstract] rtmsEcho extends the rtms framework in R for direct access to AEMS data: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
- [discussion] Peak windowing underpins rtmsEcho's automated peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] Peak detection is a critical automated task in rtmsEcho's processing: "critical tasks such as peak detection, integration, and multimodal data analysis"
