---
name: per-injection-runtime-benchmarking
description: Use when a SmartPeak CLI or GUI run has completed across a sequence of
  injections and you need per-injection wall-clock timings to quantify automated throughput
  and verify speed-up versus a manual processing baseline.
when_to_use_negative:
- The run has not finished end-to-end — export the featuredb/pivottable reports first
  to confirm completion before computing throughput.
- The goal is accuracy or integration quality (e.g. % correctly integrated features)
  rather than processing-time throughput; use feature-level QC evaluation instead.
- The dataset is not processed by SmartPeak (e.g. generic vendor-software outputs),
  since the benchmark compares SmartPeak's automated workflow against a manual processing
  baseline.
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SmartPeak
  role: Encapsulates the automated workflow (peak picking, calibration, quantitation,
    QC reporting) whose per-injection runtime is measured
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Runs the workflow from the command line with --load-session on a session directory
    and supports --report featuredb pivottable export
- name: Docker
  role: Provides the autoflowresearch/smartpeak-cli containerized environment for
    reproducible benchmark runs
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: OpenMS
  role: Underlying toolkit SmartPeak is based on, providing the algorithms executed
    during the timed workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:86cff4054b3ee24ff1286fd8e0c060e993154338488cd88d301d88aa248a79b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-injection-runtime-benchmarking

## Summary

Benchmarks SmartPeak's fully automated targeted-metabolomics data processing by recording wall-clock processing time per injection and comparing the aggregate throughput against the ~100-fold reduction reported versus manual processing.

## When to use

Use when a SmartPeak CLI or GUI run has completed across a sequence of injections and you need per-injection wall-clock timings to quantify automated throughput and verify speed-up versus a manual processing baseline.

## When NOT to use

- The run has not finished end-to-end — export the featuredb/pivottable reports first to confirm completion before computing throughput.
- The goal is accuracy or integration quality (e.g. % correctly integrated features) rather than processing-time throughput; use feature-level QC evaluation instead.
- The dataset is not processed by SmartPeak (e.g. generic vendor-software outputs), since the benchmark compares SmartPeak's automated workflow against a manual processing baseline.

## Inputs

- SmartPeak session/dataset directory containing .mzML raw data and configuration files (e.g. FIAMS_FullScan_Unknowns with parameters.csv)
- Workflow command list for the run (e.g. LOAD_RAW_DATA, EXTRACT_SPECTRA_WINDOWS, MERGE_SPECTRA, PICK_MS1_FEATURES, SEARCH_ACCURATE_MASS, STORE_ANNOTATIONS, STORE_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES, MERGE_FEATURES, MERGE_INJECTIONS, STORE_FEATURES_SAMPLE_GROUP)
- Shell or log output capturing processing time per injection

## Outputs

- Per-injection wall-clock processing time measurements (e.g. from SmartPeak log or shell timing)
- throughput_benchmark.csv with columns for injection, processing time, and automated-vs-manual speed-up ratio
- featuredb and pivottable report exports confirming completed end-to-end processing

## How to apply

Load the session from a prepared dataset directory — e.g. the FIAMS FullScan Unknowns example at src/examples/data/FIAMS_FullScan_Unknowns with its parameters.csv (acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20) — via SmartPeak CLI's --load-session, optionally inside the autoflowresearch/smartpeak-cli Docker container. Run the full FIAMS workflow (LOAD_RAW_DATA through STORE_FEATURES_SAMPLE_GROUP, including PICK_MS1_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES, MERGE_FEATURES, and MERGE_INJECTIONS) and instrument each injection's processing to capture per-injection wall-clock time, e.g. by timing shell invocations or reading SmartPeak's log output. Export featuredb and pivottable reports (--report featuredb pivottable, case-insensitive values) to confirm the workflow ran end-to-end before trusting the timings. Aggregate per-injection times into a throughput table (e.g. throughput_benchmark.csv with injection, processing time, and computed speed-up ratio columns) and check the resulting automated-vs-manual ratio is consistent with the paper's reported approximate 100-fold reduction in data processing time.

## Related tools

- **SmartPeak** (Encapsulates the automated workflow (peak picking, calibration, quantitation, QC reporting) whose per-injection runtime is measured) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Runs the workflow from the command line with --load-session on a session directory and supports --report featuredb pivottable export)
- **Docker** (Provides the autoflowresearch/smartpeak-cli containerized environment for reproducible benchmark runs) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **OpenMS** (Underlying toolkit SmartPeak is based on, providing the algorithms executed during the timed workflow)

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash && smartpeak --load-session /sample-data/FIAMS_FullScan_Unknowns --run-workflow --report featuredb pivottable
```

## Evaluation signals

- Workflow reports (featuredb, pivottable) exported with --report confirm end-to-end completion before timings are trusted
- Throughput table includes a per-injection column and a computed automated-vs-manual speed-up ratio
- Computed speed-up ratio is consistent with the reported approximate 100-fold reduction versus manual processing
- Per-injection timings are sourced from instrumented SmartPeak log or shell timing, not estimated
- Timing set covers the full sequence of injections loaded from the session directory

## Limitations

- The GUI's estimated-time display is a rough estimation only and is updated regularly while the workflow runs, so it should not be used as the timing source — instrument logs or shell timing instead.
- Speed-up depends on the specific dataset and workflow; the ~100-fold figure was reported for SmartPeak's automated processing against a manual baseline and should be re-verified per dataset.
- The automated workflow did not correctly integrate ~5–12% of features on the paper's test datasets, so throughput benchmarking should be paired with integration-accuracy checks.

## Evidence

- [readme] GUI runtime estimate is unreliable as a timing source: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset location for the benchmark session on the SmartPeak repo: [evidence span withheld — anchor status: external_doc]
