---
name: mass-spectrometry-processing-benchmark-timing
description: Use when a task asks to verify or reproduce the reported processing-speed
  benchmark for rtmsEcho or similar AEMS pipelines, e.g. timing a full 384-well plate
  full-scan (PI) run against the <30 s (up to 60 Da window) claim.
when_to_use_negative:
- The goal is to benchmark MRM-mode processing (<5 transitions), where the applicable
  benchmark is under 2 s rather than the full-scan <30 s threshold
- The data are conventional LC-MS chromatography runs rather than acoustic ejection
  mass spectrometry (AEMS) plate data, which rtmsEcho's reader and shot-to-peak association
  are not designed for
- The aim is analytical accuracy (peak areas, quantification correctness) rather than
  wall-clock throughput measurement
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: Executes the automated AEMS processing pipeline (peak detection, shot-to-peak
    association, area quantification) over all 384 wells of the full-scan plate whose
    wall-clock time is being benchmarked
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework on which rtmsEcho's example-data reader and processing
    functions are built
- name: R
  role: Runtime environment; system.time (R's system timing) captures the wall-clock
    elapsed seconds of the full-plate run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:6bfa7030595f3f7d9f6fdc191991e9970842d9abf1fa4b750388f9be81674d91
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-spectrometry-processing-benchmark-timing

## Summary

Benchmarks the wall-clock runtime of automated acoustic ejection mass spectrometry (AEMS) plate processing by timing rtmsEcho's full pipeline (peak detection, shot-to-peak association, area quantification) over a full 384-well plate and comparing the elapsed seconds against the reported benchmark of <30 s for full-scan (PI) mode with an up-to-60 Da window.

## When to use

Use when a task asks to verify or reproduce the reported processing-speed benchmark for rtmsEcho or similar AEMS pipelines, e.g. timing a full 384-well plate full-scan (PI) run against the <30 s (up to 60 Da window) claim.

## When NOT to use

- The goal is to benchmark MRM-mode processing (<5 transitions), where the applicable benchmark is under 2 s rather than the full-scan <30 s threshold
- The data are conventional LC-MS chromatography runs rather than acoustic ejection mass spectrometry (AEMS) plate data, which rtmsEcho's reader and shot-to-peak association are not designed for
- The aim is analytical accuracy (peak areas, quantification correctness) rather than wall-clock throughput measurement

## Inputs

- Bundled example 384-well plate AEMS dataset shipped with the rtmsEcho R package
- rtmsEcho full-scan (PI) mode configuration with an up-to-60 Da scan window

## Outputs

- timing_record.csv containing a single wall-clock elapsed-seconds measurement for the full 384-well plate run
- Pass/fail comparison of the measured elapsed time against the <30 s full-scan benchmark

## How to apply

Load the bundled example 384-well plate AEMS dataset shipped with the rtmsEcho R package, using the package's example-data reader built on the rtms framework. Configure rtmsEcho for full-scan precursor ion (PI) mode with the up-to-60 Da scan window from the package's full-scan configuration. Run rtmsEcho's automated pipeline (peak detection, shot-to-peak association, baseline correction, and area quantification) over all 384 wells while capturing wall-clock elapsed time with R's system.time. Write the measured elapsed seconds for the full-plate run into a single timing record (timing_record.csv) and compare it against the reported benchmark: a result under 30 s constitutes a passing check. The rationale is that rtmsEcho is claimed to process a full 384-well plate in under 2 s for MRM runs (<5 transitions) and under 30 s for full-scan mode runs (up to a 60 Da window), so timing the same bundled dataset reproduces or refutes that headline performance metric.

## Related tools

- **rtmsEcho** (Executes the automated AEMS processing pipeline (peak detection, shot-to-peak association, area quantification) over all 384 wells of the full-scan plate whose wall-clock time is being benchmarked) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework on which rtmsEcho's example-data reader and processing functions are built)
- **R** (Runtime environment; system.time (R's system timing) captures the wall-clock elapsed seconds of the full-plate run)

## Examples

```
Rscript -e 'system.time(rtmsEcho::processPlate(exampleAemsData(), mode="full-scan"))' > timing_record.csv
```

## Evaluation signals

- Recorded elapsed wall-clock time for the full 384-well full-scan (PI, up-to-60 Da window) run is less than 30 s, matching the reported benchmark
- timing_record.csv exists and contains a single numeric timing record in seconds
- The timed run covered all 384 wells of the plate, not a subset
- The processing pipeline included the automated steps (peak detection, shot-to-peak association, area quantification), not just data loading
- Timing was captured with R's system timing (e.g. system.time) so the measurement is wall-clock elapsed time

## Limitations

- The <30 s benchmark applies specifically to full-scan mode runs with an up-to-60 Da window; MRM runs (<5 transitions) use the separate <2 s benchmark
- The reported 10- to 100-fold efficiency gains over conventional methods lack named benchmark datasets or accessioned data resources, and baseline details of the conventional comparison methods are not provided
- No software version number is stated for rtmsEcho, so timing may vary across package versions
- Timing results depend on host hardware, which the benchmark as stated does not specify

## Evidence

- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] while automating shot-to-peak association and spectral analysis: "while automating shot-to-peak association and spectral analysis"
- [discussion] the package reduces processing times from hours to minutes, achieving efficiency gains of 10- to 100-fold compared to conventional methods: "the package reduces processing times from hours to minutes, achieving efficiency gains of 10- to 100-fold compared to conventional methods"
