---
name: aems-ejection-time-estimation
description: Use when AEMS .wiff raw data lacks explicit ejection times and each shot
  must be associated with its detected peak — e.g. evenly spaced acoustic ejections
  at a ~1000 ms nominal interval from a 384-well plate run.
when_to_use_negative:
- The input is conventional LC-MS chromatography with gradient retention times rather
  than acoustic ejection (AEMS) data — rtmsEcho's sinusoidal shot-to-peak association
  assumes evenly spaced ejections, and the article notes conventional chromatography
  is a fundamentally different, chromatographically separated modality.
- Explicit ejection timestamps or plate/shot metadata are already available for the
  .wiff file, making sinusoidal inference unnecessary.
- The run is not evenly spaced (e.g. irregular or aborted ejection schedules), since
  the method models ejection times as an evenly spaced set.
edam_operation: http://edamontology.org/operation_3219
tools:
- name: rtmsEcho
  role: R package that loads the AEMS .wiff data, extracts the TIC, and runs the automated
    shot-to-peak association (sinusoidal optimization) that infers ejection times
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework on which rtmsEcho is built, providing the raw mass-spectrometry
    data handling used during TIC extraction and fitting
- name: R
  role: Runtime environment for executing the rtmsEcho ejection-time inference and
    writing ejection_times.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:a6cf67f9e78b7a6c3185c50c70471394eea5e03f695fd271a9ba9d779fb4ca09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3219
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# aems-ejection-time-estimation

## Summary

Infers the precise timing of each acoustic ejection in an AEMS run (stored in AEMS .wiff files without explicit ejection timestamps) by fitting a sinusoidal reference signal to the total ion chromatogram (TIC) of total intensity versus time, achieving ~0.01 ms timing precision.

## When to use

Use when AEMS .wiff raw data lacks explicit ejection times and each shot must be associated with its detected peak — e.g. evenly spaced acoustic ejections at a ~1000 ms nominal interval from a 384-well plate run.

## When NOT to use

- The input is conventional LC-MS chromatography with gradient retention times rather than acoustic ejection (AEMS) data — rtmsEcho's sinusoidal shot-to-peak association assumes evenly spaced ejections, and the article notes conventional chromatography is a fundamentally different, chromatographically separated modality.
- Explicit ejection timestamps or plate/shot metadata are already available for the .wiff file, making sinusoidal inference unnecessary.
- The run is not evenly spaced (e.g. irregular or aborted ejection schedules), since the method models ejection times as an evenly spaced set.

## Inputs

- AEMS .wiff raw data file (acoustic ejection mass spectrometry run, e.g. from a 384-well plate)
- Start-time estimate and initial estimate of milliseconds between peaks (nominal ~1000 ms ejection interval)

## Outputs

- Table of inferred, evenly spaced ejection times written to ejection_times.csv (~0.01 ms precision)
- Fitted sinusoidal model parameters (phase/offset) matching the TIC total intensity

## How to apply

Load the AEMS .wiff raw file with the rtmsEcho package (built on the rtms framework in R) and extract the total ion chromatogram (TIC) of total signal intensity versus time. Construct a sinusoidal reference signal with a nominal period equal to the 1000 ms acoustic ejection interval. Optimize the phase/offset of the sinusoid — given a start-time estimate and an initial estimate of milliseconds between peaks — to maximize agreement between the observed TIC total intensity and the sinusoidal model, per the rtmsEcho automated shot-to-peak association algorithm. Derive the evenly spaced ejection times from the fitted sinusoid at approximately 0.01 ms precision, then write the table of estimated ejection times to ejection_times.csv.

## Related tools

- **rtmsEcho** (R package that loads the AEMS .wiff data, extracts the TIC, and runs the automated shot-to-peak association (sinusoidal optimization) that infers ejection times) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework on which rtmsEcho is built, providing the raw mass-spectrometry data handling used during TIC extraction and fitting)
- **R** (Runtime environment for executing the rtmsEcho ejection-time inference and writing ejection_times.csv)

## Evaluation signals

- The fitted ejection times are evenly spaced at the ~1000 ms nominal acoustic ejection interval.
- Residuals between the observed TIC total intensity and the fitted sinusoid are consistent with the stated ~0.01 ms peak-timing precision.
- The number of inferred ejection times matches the expected number of wells/shots in the plate run (e.g. 384 for a 384-well plate).
- Each inferred ejection time associates to exactly one detected peak in the TIC (successful automated shot-to-peak association).
- ejection_times.csv is produced with a time column whose units and ordering (first ejection, start-time offset) match the run metadata.

## Limitations

- Ejection times are not explicitly stored in AEMS .wiff files; the method must infer them, so a poor start-time estimate or wrong initial ms-between-peaks estimate can mislead the sinusoidal optimization.
- The model assumes ejection times form an evenly spaced set at a single nominal interval; deviations from even spacing are not modeled.
- rtmsEcho's speed claims (full 384-well plate processed in <2 s for MRM, <30 s for full-scan) and 10- to 100-fold efficiency gains are not backed by named benchmark datasets in the provided context, and no software version or changelog is stated.

## Evidence

- [full_text] Sinusoidal TIC fit identifies ejection timing: [evidence span withheld — anchor status: not_found]
- [full_text] Evenly spaced inference from start-time and interval estimates: [evidence span withheld — anchor status: not_found]
- [abstract] Automated shot-to-peak association: "while automating shot-to-peak association and spectral analysis"
- [abstract] Direct AEMS .wiff access via rtmsEcho: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions"
