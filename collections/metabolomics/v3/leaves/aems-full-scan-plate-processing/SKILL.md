---
name: aems-full-scan-plate-processing
description: Use when a full 384-well plate of AEMS data in full-scan (PI) mode must
  be processed end-to-end with rtmsEcho and the elapsed wall-clock time must be verified
  against the reported <30 s (up to a 60 Da window) benchmark.
when_to_use_negative:
- The run is MRM mode rather than full-scan/PI — the package reports a different (<2
  s, <5 transitions) benchmark and a different processing path for MRM.
- The input is not a 384-well AEMS plate dataset loadable by the rtmsEcho example-data
  reader built on the rtms framework (e.g. conventional LC-MS chromatography data).
- The goal is qualitative data inspection rather than a timed end-to-end processing
  benchmark.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: Runs the automated full-scan (PI) AEMS plate processing pipeline — peak detection,
    shot-to-peak association, area quantification — over all 384 wells; its bundled
    example dataset supplies the input plate.
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying mass-spectrometry framework on which rtmsEcho's example-data reader
    and processing functions are built.
- name: R
  role: Execution environment; system timing (e.g. system.time) is used to capture
    wall-clock elapsed seconds for the full-plate run.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:178c073177074ce18a4679058fb41cc6d45b2de97c087b6e32d8573fd05126a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# aems-full-scan-plate-processing

## Summary

Benchmarks the wall-clock processing time of rtmsEcho (an open-source R package built on the rtms framework) when running its automated pipeline — peak detection, shot-to-peak association, and area quantification — over a full 384-well acoustic ejection mass spectrometry (AEMS) plate in full-scan precursor ion (PI) mode, checking against the paper's reported benchmark of <30 s for a scan window up to 60 Da.

## When to use

Use when a full 384-well plate of AEMS data in full-scan (PI) mode must be processed end-to-end with rtmsEcho and the elapsed wall-clock time must be verified against the reported <30 s (up to a 60 Da window) benchmark.

## When NOT to use

- The run is MRM mode rather than full-scan/PI — the package reports a different (<2 s, <5 transitions) benchmark and a different processing path for MRM.
- The input is not a 384-well AEMS plate dataset loadable by the rtmsEcho example-data reader built on the rtms framework (e.g. conventional LC-MS chromatography data).
- The goal is qualitative data inspection rather than a timed end-to-end processing benchmark.

## Inputs

- Bundled example 384-well plate AEMS dataset distributed with the rtmsEcho R package
- Full-scan (precursor ion / PI) mode configuration with scan window up to 60 Da

## Outputs

- timing_record.csv containing a single timing record with the measured elapsed seconds for the full-plate run
- Processed per-well peak results (peak detection, shot-to-peak association, area quantification) for all 384 wells

## How to apply

Load the bundled example 384-well plate AEMS dataset shipped with the rtmsEcho R package (github.com/nathanieltwarog/rtmsEcho), which extends the previously published rtms framework. Configure rtmsEcho for full-scan precursor ion (PI) mode using the up-to-60 Da scan window from the package's full-scan configuration — not MRM, PI and NL are among the supported scan modes but this skill targets the full-scan PI path. Execute the automated processing pipeline (peak detection, shot-to-peak association, and area quantification across all 384 wells) while capturing wall-clock elapsed time with R's system timing (e.g. system.time). Write the measured elapsed seconds into a single timing record saved as timing_record.csv, then validate: a result under 30 s constitutes a passing check against the reported benchmark.

## Related tools

- **rtmsEcho** (Runs the automated full-scan (PI) AEMS plate processing pipeline — peak detection, shot-to-peak association, area quantification — over all 384 wells; its bundled example dataset supplies the input plate.) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying mass-spectrometry framework on which rtmsEcho's example-data reader and processing functions are built.)
- **R** (Execution environment; system timing (e.g. system.time) is used to capture wall-clock elapsed seconds for the full-plate run.)

## Evaluation signals

- Recorded wall-clock elapsed time in timing_record.csv is under 30 s, matching the reported benchmark for full-scan (PI) mode runs with an up-to-60 Da window.
- The timing record covers processing of all 384 wells, not a subset.
- The pipeline ran in full-scan PI mode (not MRM or NL) with the scan window at or below 60 Da.
- All three automated stages completed: peak detection, shot-to-peak association, and area quantification.
- timing_record.csv is a single timing record with the elapsed-seconds measurement, per the sub-task's output specification.

## Limitations

- The <30 s benchmark applies specifically to full-scan mode runs with an up-to-60 Da scan window; MRM runs are benchmarked separately (<2 s for <5 transitions).
- No software version number is stated for rtmsEcho (no changelog found), so reproducibility of the timing measurement depends on the installed build.
- No benchmark datasets or accessioned data resources are named to support the reported 10- to 100-fold speedup over conventional methods, and baseline details of those conventional methods are not provided.

## Evidence

- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes): "enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
- [abstract] while automating shot-to-peak association and spectral analysis: "while automating shot-to-peak association and spectral analysis"
