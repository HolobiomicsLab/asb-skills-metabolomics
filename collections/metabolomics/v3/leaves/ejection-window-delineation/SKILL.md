---
name: ejection-window-delineation
description: Use when you have an AEMS run's ejection-time table plus a TIC trace
  and must assign each acoustic ejection a bounded detection window with per-ejection
  peak timing.
when_to_use_negative:
- Input is a conventional LC-MS chromatogram rather than an acoustic-ejection (AEMS)
  run of discrete ejections from a 384- or 1536-well plate
- The upstream ejection-time table is not yet available, since this step consumes
  those times to bracket windows between adjacent ejections
- You need peak integration or area quantification itself rather than the timing-window
  delineation that precedes them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rtmsEcho
  role: R package that smooths the TIC trace, finds local minima between adjacent
    ejection times, and applies the background-noise threshold to produce per-ejection
    windows
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: underlying R framework extended by rtmsEcho, used to access and extract the
    TIC trace from raw AEMS data
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: R
  role: language in which the rtmsEcho smoothing, minima-finding, and windowing workflow
    is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:dd0ef44678995b28c8b691a1f646994bb7abb2afe8d5cb5dc5f57f7f58236455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ejection-window-delineation

## Summary

Delineates the detection window for each acoustic ejection in an AEMS run by lightly smoothing the TIC intensity trace, locating local minima between adjacent ejection times, and applying a background-noise threshold, yielding per-ejection peak_time and min_time/max_time bounds.

## When to use

Use when you have an AEMS run's ejection-time table plus a TIC trace and must assign each acoustic ejection a bounded detection window with per-ejection peak timing.

## When NOT to use

- Input is a conventional LC-MS chromatogram rather than an acoustic-ejection (AEMS) run of discrete ejections from a 384- or 1536-well plate
- The upstream ejection-time table is not yet available, since this step consumes those times to bracket windows between adjacent ejections
- You need peak integration or area quantification itself rather than the timing-window delineation that precedes them

## Inputs

- ejection-time table produced by the upstream timing-inference step for an AEMS run
- TIC (total ion current) intensity trace extracted from raw AEMS data (MRM or full-scan/PI/NL acquisitions)
- estimated background-noise level of the TIC trace

## Outputs

- per-ejection window records containing peak_time, min_time, and max_time
- ejection_windows.csv table of accepted ejection windows

## How to apply

Load the ejection-time table from the upstream timing-inference step together with the TIC trace extracted from the raw AEMS data via the rtmsEcho/rtms framework in R. Smooth the TIC signal with a local smoothing filter so shot-to-shot fluctuations do not create spurious minima. For each pair of adjacent ejection times, scan the smoothed trace within the intervening interval for the local minima bracketing the expected peak; set window boundaries where total intensity falls to the background-noise threshold, or at the smoothed-trace local minimum if intensity never drops below that threshold between adjacent ejections. Reject windows whose signal does not rise above the background-noise level, then record peak_time (local maximum within the window) and min_time/max_time (flanking minima) for each accepted ejection and write the records as ejection_windows.csv.

## Related tools

- **rtmsEcho** (R package that smooths the TIC trace, finds local minima between adjacent ejection times, and applies the background-noise threshold to produce per-ejection windows) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (underlying R framework extended by rtmsEcho, used to access and extract the TIC trace from raw AEMS data) — https://github.com/nathanieltwarog/rtmsEcho
- **R** (language in which the rtmsEcho smoothing, minima-finding, and windowing workflow is executed)

## Evaluation signals

- Each accepted ejection yields exactly one peak_time that lies between its min_time and max_time
- Windows whose signal never exceeds the background-noise threshold are excluded from ejection_windows.csv
- Window bounds (min_time/max_time) coincide with local minima of the smoothed TIC trace between adjacent ejection times
- Every ejection in the upstream ejection-time table maps to at most one window record, with noise-only windows rejected
- The full output is assembled as ejection_windows.csv with one record per accepted ejection

## Limitations

- Windows between adjacent ejections where intensity never falls below the background-noise threshold must fall back to the smoothed-trace local minimum, which may overstate the true detection window
- Without prior smoothing, shot-to-shot fluctuations in the TIC create spurious local minima and malformed windows
- Ejections whose signal does not rise above background noise cannot be windowed and are dropped entirely

## Evidence

- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions"
- [abstract] shot-to-peak association: "while automating shot-to-peak association and spectral analysis"
- [discussion] peak alignment/baseline correction/area quantification automation: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [intro] processing speed: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs"
