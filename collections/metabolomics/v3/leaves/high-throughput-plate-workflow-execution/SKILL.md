---
name: high-throughput-plate-workflow-execution
description: Use when validating or benchmarking rtmsEcho's end-to-end plate-level
  processing throughput, e.g. timing a full 384-well plate AEMS run in full-scan precursor
  ion (PI) mode to check the <30 s claim.
when_to_use_negative:
- The goal is analytical output quality (peak areas, identifications) rather than
  measuring or validating processing throughput
- The data are conventional LC-MS runs with chromatographic separation rather than
  acoustic ejection mass spectrometry (AEMS) plate data
- The plate has more wells (e.g. 1536-well) or a full-scan window wider than 60 Da,
  since the reported <30 s benchmark applies only to 384-well plates with up-to-60
  Da windows
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rtmsEcho
  role: Runs the automated AEMS processing pipeline (peak detection, shot-to-peak
    association, area quantification) over all wells of the 384-well plate; its bundled
    example data and full-scan configuration supply the inputs for the timed run
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework that rtmsEcho extends, providing direct access to raw
    AEMS data for the package's example-data reader
  repo: github.com/nathanieltwarog/rtmsEcho
- name: R
  role: Host language for executing the pipeline and capturing wall-clock elapsed
    time via R's system timing (e.g. system.time)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:bd1fad4fe3bd437c196d8426dd9a9e8dc9a396b152d1d2bcae36179c86c4ba8e
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# high-throughput-plate-workflow-execution

## Summary

Execute rtmsEcho's automated acoustic ejection mass spectrometry (AEMS) pipeline over all wells of a 384-well plate and measure wall-clock processing time against the reported benchmarks (<2 s for MRM with <5 transitions; <30 s for full-scan mode up to a 60 Da window).

## When to use

Use when validating or benchmarking rtmsEcho's end-to-end plate-level processing throughput, e.g. timing a full 384-well plate AEMS run in full-scan precursor ion (PI) mode to check the <30 s claim.

## When NOT to use

- The goal is analytical output quality (peak areas, identifications) rather than measuring or validating processing throughput
- The data are conventional LC-MS runs with chromatographic separation rather than acoustic ejection mass spectrometry (AEMS) plate data
- The plate has more wells (e.g. 1536-well) or a full-scan window wider than 60 Da, since the reported <30 s benchmark applies only to 384-well plates with up-to-60 Da windows

## Inputs

- Bundled example 384-well plate AEMS dataset from the rtmsEcho R package
- Plate and compound metadata associated with the raw AEMS data
- Mode configuration: full-scan precursor ion (PI) mode with up-to-60 Da scan window, or MRM transition list (<5 transitions)

## Outputs

- timing_record.csv containing a single wall-clock elapsed-seconds record for the full-plate run
- Per-well processed results (peak detection, shot-to-peak association, area quantification) from the rtmsEcho pipeline

## How to apply

Load the bundled example 384-well plate AEMS dataset shipped with the rtmsEcho R package (github.com/nathanieltwarog/rtmsEcho), which builds on the rtms framework for direct access to AEMS data. Configure the pipeline for the target acquisition mode — full-scan precursor ion (PI) mode with the package's up-to-60 Da scan window, or MRM with fewer than 5 transitions for the fast benchmark. Run rtmsEcho's automated processing (peak detection, shot-to-peak association, peak alignment, baseline correction, and area quantification) over all 384 wells while wrapping the call in R's system timing (e.g. system.time or proc.time) to capture elapsed seconds. Write the measured elapsed time into a single timing record (timing_record.csv) and compare it to the reported benchmark: <30 s for full-scan mode runs (up to a 60 Da window), <2 s for MRM runs (<5 transitions). A wall-clock result under the relevant threshold constitutes a passing check; results far above it indicate either misconfiguration (wrong mode or window) or a regression against the published 10- to 100-fold efficiency gains over conventional methods.

## Related tools

- **rtmsEcho** (Runs the automated AEMS processing pipeline (peak detection, shot-to-peak association, area quantification) over all wells of the 384-well plate; its bundled example data and full-scan configuration supply the inputs for the timed run) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework that rtmsEcho extends, providing direct access to raw AEMS data for the package's example-data reader) — github.com/nathanieltwarog/rtmsEcho
- **R** (Host language for executing the pipeline and capturing wall-clock elapsed time via R's system timing (e.g. system.time))

## Evaluation signals

- Recorded elapsed wall-clock time for the full 384-well plate full-scan (PI) run is under 30 s (benchmark applies up to a 60 Da window)
- For MRM mode, elapsed time for a full 384-well plate with <5 transitions is under 2 s
- timing_record.csv exists and contains a single elapsed-seconds timing record for the full-plate run
- All 384 wells were processed by the pipeline (peak detection, shot-to-peak association, and area quantification completed without errors)
- Observed throughput is consistent with the reported 10- to 100-fold efficiency gain over conventional methods (hours to minutes)

## Limitations

- The <30 s benchmark is specific to full-scan mode runs with an up-to-60 Da window; larger windows or other configurations are not covered
- No software version number is stated for rtmsEcho and no benchmark datasets or accessioned data resources were named to support the reported 10- to 100-fold speedup, so the comparison baseline is not fully specified
- Existing open-source MRM data processing platforms do not address AEMS's specialized data architecture, so timing comparisons must use rtmsEcho itself rather than generic MS tools

## Evidence

- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] we developed rtmsEcho, an open-source R package that extends our previously published rtms framework: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes): "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
