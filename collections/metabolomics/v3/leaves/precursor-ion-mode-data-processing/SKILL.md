---
name: precursor-ion-mode-data-processing
description: Use when AEMS plate data was acquired in full-scan precursor ion (PI)
  mode and all wells of a 384-well plate must be processed (peak detection, shot-to-peak
  association, area quantification) or benchmarked against the reported <30 s runtime.
when_to_use_negative:
- The acquisition is MRM mode with <5 transitions — the applicable benchmark is under
  2 s per 384-well plate, not the <30 s full-scan figure
- The data are conventional LC-MS with chromatographic separation rather than AEMS
  plate data; rtmsEcho is specialized for AEMS's chromatography-free, ultrahigh-throughput
  data architecture
- Only precursor-ion or neutral-loss mode coverage of other scan modes (e.g. NL) is
  needed and the timing benchmark for PI mode is irrelevant to the question at hand
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rtmsEcho
  role: Primary R package that performs the automated full-scan PI-mode processing
    of the 384-well AEMS plate (peak detection, shot-to-peak association, area quantification)
    and supplies the bundled example dataset and full-scan configuration
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework on which rtmsEcho is built; provides the data-access
    layer used by the example-data reader for loading AEMS plate data
- name: R
  role: Execution environment in which rtmsEcho is run and wall-clock timing is captured
    via R's system timing (e.g. system.time) before writing timing_record.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:a553c5eb56eeb6db8112d06d9b6b03ce4741f8aeffa03b7ed4c7feb08effce6f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# precursor-ion-mode-data-processing

## Summary

Processes full 384-well acoustic ejection mass spectrometry (AEMS) plates acquired in full-scan precursor ion (PI) mode using the rtmsEcho R package, and verifies wall-clock performance against the reported benchmark of under 30 s per plate for scan windows up to 60 Da.

## When to use

Use when AEMS plate data was acquired in full-scan precursor ion (PI) mode and all wells of a 384-well plate must be processed (peak detection, shot-to-peak association, area quantification) or benchmarked against the reported <30 s runtime.

## When NOT to use

- The acquisition is MRM mode with <5 transitions — the applicable benchmark is under 2 s per 384-well plate, not the <30 s full-scan figure
- The data are conventional LC-MS with chromatographic separation rather than AEMS plate data; rtmsEcho is specialized for AEMS's chromatography-free, ultrahigh-throughput data architecture
- Only precursor-ion or neutral-loss mode coverage of other scan modes (e.g. NL) is needed and the timing benchmark for PI mode is irrelevant to the question at hand

## Inputs

- 384-well plate AEMS raw data (rtmsEcho bundled example dataset readable through the package's example-data reader built on the rtms framework)
- Full-scan precursor ion (PI) mode processing configuration with an up-to-60 Da scan window
- Plate and compound metadata associated with the raw data upload

## Outputs

- timing_record.csv containing a single wall-clock elapsed-seconds record for the full 384-well plate run
- Per-well automated processing results from peak detection, shot-to-peak association, and area quantification

## How to apply

Load the bundled example 384-well plate AEMS dataset shipped with the rtmsEcho R package via the package's example-data reader built on the rtms framework. Configure rtmsEcho for full-scan precursor ion (PI) mode, using the up-to-60 Da scan window from the package's full-scan configuration — the 60 Da window is the scope under which the <30 s benchmark holds. Execute rtmsEcho's automated pipeline over all 384 wells; the pipeline automates peak detection, peak alignment, baseline correction, shot-to-peak association, and area quantification, which are the steps existing AEMS tools lack. Capture wall-clock elapsed time with R's system timing (e.g. system.time) and write the measured elapsed seconds into a single timing record saved as timing_record.csv. The run passes if elapsed wall-clock time for the full-plate full-scan (PI) processing is under 30 s.

## Related tools

- **rtmsEcho** (Primary R package that performs the automated full-scan PI-mode processing of the 384-well AEMS plate (peak detection, shot-to-peak association, area quantification) and supplies the bundled example dataset and full-scan configuration) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework on which rtmsEcho is built; provides the data-access layer used by the example-data reader for loading AEMS plate data)
- **R** (Execution environment in which rtmsEcho is run and wall-clock timing is captured via R's system timing (e.g. system.time) before writing timing_record.csv)

## Evaluation signals

- Recorded wall-clock elapsed time for the full 384-well plate full-scan (PI) run is under 30 s, matching the reported benchmark
- The PI-mode configuration uses a scan window no larger than 60 Da, the scope for which the <30 s benchmark is stated
- timing_record.csv contains exactly one timing record with elapsed seconds for the full-plate run
- The automated pipeline ran over all 384 wells, completing peak detection, shot-to-peak association, and area quantification
- The run is reproducible from the bundled example 384-well AEMS dataset shipped with the package

## Limitations

- The <30 s benchmark applies to full-scan mode runs with scan windows up to 60 Da; wider windows or larger plates may exceed it
- No benchmark datasets or accessioned data resources are named to support the reported 10- to 100-fold speedup, and baseline details of the conventional methods used for comparison are not provided
- No software version number or changelog is stated for rtmsEcho, so reproduced timings may vary with package version

## Evidence

- [intro] Reported <30 s full-scan benchmark: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] PI-mode scope of rtmsEcho: "enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] Automated processing steps executed over the plate: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [discussion] Scan-mode support: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
