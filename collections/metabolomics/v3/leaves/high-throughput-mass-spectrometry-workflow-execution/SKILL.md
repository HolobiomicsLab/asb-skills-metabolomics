---
name: high-throughput-mass-spectrometry-workflow-execution
description: Use when timing how fast rtmsEcho processes a full 384-well MRM AEMS
  plate (<5 transitions) and the measured wall-clock duration must be validated against
  the reported <2 s benchmark.
when_to_use_negative:
- The acquisition is full-scan mode (up to a 60 Da window), where the applicable benchmark
  is <30 s, not <2 s
- The plate is not a full 384-well plate or the MRM run has 5 or more transitions,
  so the reported <2 s figure does not apply
- The goal is processing with vendor-specific AEMS software rather than measuring
  rtmsEcho's open-source pipeline performance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: Executes the automated MRM processing pipeline (peak detection, shot-to-peak
    association, area quantification) over the 384-well plate whose runtime is being
    benchmarked
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R mass-spectrometry framework that rtmsEcho extends and on which
    the processing routines run
- name: R
  role: Host environment in which the processing call is wrapped in system-time wall-clock
    measurement and results are serialized to JSON
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:180a0ff11de78e20613f68ab768d6a0354f9108aa9f8eba6ba526bbd5fd3f71d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# high-throughput-mass-spectrometry-workflow-execution

## Summary

Benchmark skill that executes the rtmsEcho MRM processing pipeline over a full 384-well acoustic ejection mass spectrometry (AEMS) plate and measures wall-clock processing time against the paper's reported <2 s benchmark for MRM runs with fewer than 5 transitions.

## When to use

Use when timing how fast rtmsEcho processes a full 384-well MRM AEMS plate (<5 transitions) and the measured wall-clock duration must be validated against the reported <2 s benchmark.

## When NOT to use

- The acquisition is full-scan mode (up to a 60 Da window), where the applicable benchmark is <30 s, not <2 s
- The plate is not a full 384-well plate or the MRM run has 5 or more transitions, so the reported <2 s figure does not apply
- The goal is processing with vendor-specific AEMS software rather than measuring rtmsEcho's open-source pipeline performance

## Inputs

- Bundled example 384-well MRM plate data distributed with the rtmsEcho R package
- Plate and compound metadata associated with the raw AEMS data files
- MRM transition list (fewer than 5 transitions, per the benchmark's scope)

## Outputs

- mrm_processing_time.json containing the elapsed wall-clock seconds for the full 384-well run and the comparison against the <2 s benchmark

## How to apply

Load the bundled example 384-well MRM plate data shipped with the rtmsEcho R package via its example-data accessor. Invoke the rtmsEcho MRM processing routine — which performs peak detection, shot-to-peak association, peak alignment, baseline correction, and area quantification on the underlying rtms framework — across the full plate, ensuring the acquisition has fewer than 5 MRM transitions since the <2 s benchmark applies only to that regime. Wrap the processing call in R's system-time wall-clock measurement to capture elapsed seconds for the complete run. Compare the measured elapsed time to the <2 s threshold and record both the elapsed seconds and the pass/fail comparison in mrm_processing_time.json. The rationale is that rtmsEcho was explicitly engineered for ultrahigh-throughput AEMS data where vendor-specific software is suboptimal in throughput, so the timing itself is a claim about the package's core performance.

## Related tools

- **rtmsEcho** (Executes the automated MRM processing pipeline (peak detection, shot-to-peak association, area quantification) over the 384-well plate whose runtime is being benchmarked) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R mass-spectrometry framework that rtmsEcho extends and on which the processing routines run)
- **R** (Host environment in which the processing call is wrapped in system-time wall-clock measurement and results are serialized to JSON)

## Evaluation signals

- The recorded elapsed wall-clock time for the full 384-well MRM plate is less than 2 s
- The timed run covers the complete 384-well plate, not a subset of wells
- The MRM acquisition contains fewer than 5 transitions, matching the benchmark's stated scope
- mrm_processing_time.json contains both the elapsed seconds and the explicit benchmark comparison
- The timing is reproducible across repeated runs within a margin consistent with the <2 s claim

## Limitations

- The <2 s figure applies only to MRM runs with fewer than 5 transitions; full-scan mode runs (up to a 60 Da window) use a separate <30 s benchmark
- No software version number for rtmsEcho is stated, and no benchmark datasets or accessioned data resources are named to support the reported 10- to 100-fold speedup over conventional methods
- Measured wall-clock time is machine-dependent, so the benchmark should be interpreted on hardware comparable to that used in the paper
- Baseline details of the conventional methods used for comparison are not provided, so relative-speed claims cannot be independently verified from the article alone

## Evidence

- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] By streamlining data extraction and quantification, rtmsEcho enhances efficiency and reproducibility in high-throughput applications: "By streamlining data extraction and quantification, rtmsEcho enhances efficiency and reproducibility in high-throughput applications"
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
