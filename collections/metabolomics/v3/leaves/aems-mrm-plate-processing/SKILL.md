---
name: aems-mrm-plate-processing
description: Use when you must process a 384-well AEMS plate in MRM mode (<5 transitions)
  with rtmsEcho and check whether the sub-2 s per-plate processing benchmark reported
  in the paper holds on your run.
when_to_use_negative:
- The acquisition is full-scan AEMS data (PI or NL modes, up to a 60 Da window) —
  the applicable benchmark there is under 30 s, not the MRM <2 s figure
- The MRM run has 5 or more transitions, so the article's <2 s per-384-well benchmark
  no longer applies
- The data are conventional chromatography-coupled LC-MS runs rather than acoustic
  ejection mass spectrometry plates — rtmsEcho is specialized to AEMS data architectures
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: 'R package that performs the full MRM plate processing: raw data upload with
    plate and compound metadata, automated peak detection, peak alignment, baseline
    correction, shot-to-peak association, and area quantification'
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework that rtmsEcho extends and builds on for the MRM processing
    and area quantification
- name: R
  role: Runtime environment in which rtmsEcho is invoked and system-time wall-clock
    timing of the 384-well run is captured
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:82044b1b34d7c47938f87a3eef3f5ba43f37e5ac4c694ede546b01d53f8cf017
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# aems-mrm-plate-processing

## Summary

Processes a full 384-well acoustic ejection mass spectrometry (AEMS) plate in MRM mode with rtmsEcho (peak detection, shot-to-peak association, and area quantification built on the rtms framework), and verifies the paper's benchmark of under 2 s wall-clock processing for plates with fewer than 5 MRM transitions.

## When to use

Use when you must process a 384-well AEMS plate in MRM mode (<5 transitions) with rtmsEcho and check whether the sub-2 s per-plate processing benchmark reported in the paper holds on your run.

## When NOT to use

- The acquisition is full-scan AEMS data (PI or NL modes, up to a 60 Da window) — the applicable benchmark there is under 30 s, not the MRM <2 s figure
- The MRM run has 5 or more transitions, so the article's <2 s per-384-well benchmark no longer applies
- The data are conventional chromatography-coupled LC-MS runs rather than acoustic ejection mass spectrometry plates — rtmsEcho is specialized to AEMS data architectures

## Inputs

- Full 384-well AEMS MRM acquisition data with fewer than 5 transitions (bundled example plate data shipped with the rtmsEcho package)
- Plate metadata associating wells with shots/samples
- Compound metadata defining the targeted MRM transitions

## Outputs

- Processed peak detections with shot-to-peak associations for all 384 wells
- Area quantification (integrated peak areas with baseline correction) per transition per well
- mrm_processing_time.json recording elapsed wall-clock seconds for the full-plate run and the comparison against the <2 s benchmark

## How to apply

Load the bundled example 384-well MRM plate data shipped with rtmsEcho via its example-data accessor in R, along with the associated plate and compound metadata. Invoke the rtmsEcho MRM processing routine, which automates peak detection, peak alignment, baseline correction, shot-to-peak association, and area quantification on top of the rtms framework. Wrap the processing call in R's system-time wall-clock measurement to capture elapsed seconds for the complete 384-well run. The decision threshold is the article's reported benchmark: a full 384-well plate in under 2 s for MRM runs with fewer than 5 transitions (in contrast, full-scan mode runs with up to a 60 Da window are benchmarked at under 30 s). Record the measured elapsed seconds and the benchmark comparison into mrm_processing_time.json; a passing run is one where the recorded wall-clock time is below 2 s.

## Related tools

- **rtmsEcho** (R package that performs the full MRM plate processing: raw data upload with plate and compound metadata, automated peak detection, peak alignment, baseline correction, shot-to-peak association, and area quantification) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework that rtmsEcho extends and builds on for the MRM processing and area quantification)
- **R** (Runtime environment in which rtmsEcho is invoked and system-time wall-clock timing of the 384-well run is captured)

## Evaluation signals

- Recorded elapsed wall-clock time in mrm_processing_time.json is less than 2 s, matching the article's reported processing speed for MRM runs with fewer than 5 transitions
- The processed run covers all 384 wells of the plate, not a subset
- The acquisition is confirmed to be MRM mode with fewer than 5 transitions before applying the <2 s threshold
- The processing pipeline includes the article's automated steps — peak detection, shot-to-peak association, baseline correction, and area quantification — not just raw data import
- The output JSON contains both the elapsed seconds and an explicit benchmark comparison field

## Limitations

- The <2 s benchmark is specific to MRM runs with fewer than 5 transitions; full-scan mode runs (up to a 60 Da window) are benchmarked separately at under 30 s
- The reported 10- to 100-fold efficiency gains over conventional methods are not backed by named benchmark datasets or accessioned data resources in the article
- No software version number for rtmsEcho is stated, so timing results may not be reproducible against a pinned release

## Evidence

- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes): "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] we developed rtmsEcho, an open-source R package that extends our previously published rtms framework: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
