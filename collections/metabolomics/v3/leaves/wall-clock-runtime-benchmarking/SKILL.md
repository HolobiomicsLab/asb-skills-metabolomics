---
name: wall-clock-runtime-benchmarking
description: Use when reproducing or verifying rtmsEcho's reported processing-speed
  benchmark on acoustic ejection mass spectrometry (AEMS) data — specifically, timing
  a complete 384-well MRM plate (<5 transitions) against the reported <2 s threshold.
when_to_use_negative:
- The acquisition is full-scan mode (precursor ion / neutral loss, up to a 60 Da window)
  rather than MRM — the applicable benchmark is then <30 s per 384-well plate, not
  <2 s
- The goal is a speedup comparison against conventional AEMS processing methods —
  the article names no benchmark datasets or baseline details to support its reported
  10- to 100-fold efficiency-gain claim
- The input is a partial run or sub-plate rather than a complete 384-well plate, so
  the reported full-plate timing threshold does not directly apply
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rtmsEcho
  role: 'Target software under benchmarking: its automated MRM processing pipeline
    (peak detection, shot-to-peak association, area quantification) is the code path
    whose wall-clock runtime is measured'
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework on which rtmsEcho's MRM processing routine is built;
    part of the code path being timed
- name: R
  role: Language used to load the example 384-well plate, run the processing call,
    and capture elapsed wall-clock time via system-time measurement
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:c289d0bc81e5175bc9ed0bf6ac32d2586d5af520174cd9348dc3e1520fccc51e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# wall-clock-runtime-benchmarking

## Summary

Measures end-to-end wall-clock runtime of rtmsEcho's automated AEMS processing pipeline over a full 384-well plate to verify the paper's benchmark: <2 s for MRM runs with <5 transitions (vs. <30 s for full-scan runs up to a 60 Da window).

## When to use

Use when reproducing or verifying rtmsEcho's reported processing-speed benchmark on acoustic ejection mass spectrometry (AEMS) data — specifically, timing a complete 384-well MRM plate (<5 transitions) against the reported <2 s threshold.

## When NOT to use

- The acquisition is full-scan mode (precursor ion / neutral loss, up to a 60 Da window) rather than MRM — the applicable benchmark is then <30 s per 384-well plate, not <2 s
- The goal is a speedup comparison against conventional AEMS processing methods — the article names no benchmark datasets or baseline details to support its reported 10- to 100-fold efficiency-gain claim
- The input is a partial run or sub-plate rather than a complete 384-well plate, so the reported full-plate timing threshold does not directly apply

## Inputs

- Bundled example 384-well MRM plate data shipped with the rtmsEcho R package (accessed via the package's example-data accessor)
- MRM acquisition with fewer than 5 transitions per run

## Outputs

- mrm_processing_time.json recording the measured elapsed wall-clock seconds for the full 384-well plate and the comparison against the <2 s benchmark
- Binary pass/fail verdict: elapsed time < 2 s

## How to apply

Load the bundled example 384-well MRM plate data shipped with the rtmsEcho package via its example-data accessor in R. Invoke rtmsEcho's MRM processing routine — peak detection, shot-to-peak association, and area quantification built on the rtms framework — over the entire plate in a single call. Wrap that call in R's system-time wall-clock measurement (system.time / proc.time) to capture elapsed seconds for the complete run. Compare the measured elapsed time against the reported benchmark of less than 2 s for a full 384-well MRM plate with fewer than 5 transitions. Write the elapsed seconds and the benchmark comparison into mrm_processing_time.json; the recorded time passing under 2 s is the validation criterion.

## Related tools

- **rtmsEcho** (Target software under benchmarking: its automated MRM processing pipeline (peak detection, shot-to-peak association, area quantification) is the code path whose wall-clock runtime is measured) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework on which rtmsEcho's MRM processing routine is built; part of the code path being timed)
- **R** (Language used to load the example 384-well plate, run the processing call, and capture elapsed wall-clock time via system-time measurement)

## Evaluation signals

- The recorded elapsed wall-clock time for the full 384-well MRM plate is less than 2 seconds, matching the article's reported processing speed for MRM runs with fewer than 5 transitions
- The timed run covers the complete plate (all 384 wells) and fewer than 5 MRM transitions, matching the benchmark's stated scope
- mrm_processing_time.json contains both the numeric elapsed-seconds value and an explicit benchmark comparison
- Timing is captured as wall-clock time around the single end-to-end processing call (peak detection + shot-to-peak association + area quantification), not around individual sub-steps

## Limitations

- The <2 s threshold applies only to MRM runs with <5 transitions; full-scan mode runs (up to a 60 Da window) use a separate <30 s benchmark
- No benchmark datasets or accessioned data resources are named to support the reported 10- to 100-fold speedup over conventional methods, and baseline details of those conventional methods are not provided
- No software version number is stated for rtmsEcho, and no DOI or venue is given at the repository, complicating exact reproducibility of the timing environment
- Wall-clock timings are machine-dependent; the verification is a threshold check against the paper's hardware, not an absolute performance guarantee

## Evidence

- [intro] 384-well MRM benchmark threshold: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] claimed speedup over conventional methods: "the package reduces processing times from hours to minutes, achieving efficiency gains of 10- to 100-fold compared to conventional methods"
- [discussion] absence of named benchmark datasets: [evidence span withheld — anchor status: not_found]
