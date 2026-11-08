---
name: automated-vs-manual-speedup-comparison
description: Use when you have a runnable automated pipeline for targeted/untargeted
  metabolomics (e.g. SmartPeak on mzML injections) and need to measure how much faster
  it processes a sequence than manual analyst-driven processing.
when_to_use_negative:
- You want to measure analytical throughput of the instrument acquisition itself rather
  than software data-processing speed
- The pipeline has no runnable example dataset or parameters.csv, so only a qualitative
  claim of speed-up would be possible
- The comparison targets accuracy or peak-integration quality rather than processing-time
  reduction
tools:
- name: SmartPeak
  role: Application whose automated workflow is timed per injection to produce the
    speed-up measurement
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line entry point used to load the session and run the workflow with
    instrumentation
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: OpenMS
  role: Underlying toolkit providing the peak-picking and feature-detection algorithms
    being timed
- name: Docker
  role: Container runtime used to launch the smartpeak-cli container for a reproducible
    timed run
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:6424e32759f8d16a26d03c015b7dd33cc1cf89f479c006343fe08b584ee0e5a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# automated-vs-manual-speedup-comparison

## Summary

Quantifies the wall-clock speed-up of a fully automated metabolomics data-processing workflow (e.g. SmartPeak on OpenMS) against a manual baseline, using per-injection timing instrumentation on a runnable example dataset.

## When to use

Use when you have a runnable automated pipeline for targeted/untargeted metabolomics (e.g. SmartPeak on mzML injections) and need to measure how much faster it processes a sequence than manual analyst-driven processing.

## When NOT to use

- You want to measure analytical throughput of the instrument acquisition itself rather than software data-processing speed
- The pipeline has no runnable example dataset or parameters.csv, so only a qualitative claim of speed-up would be possible
- The comparison targets accuracy or peak-integration quality rather than processing-time reduction

## Inputs

- Session directory with .mzML raw injection files and sequence table (e.g. SmartPeak FIAMS_FullScan_Unknowns example)
- parameters.csv workflow configuration with acquisition and scan settings (e.g. acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20)
- Manual processing time baseline for the same or comparable injection volume (from the analyst workflow being compared)
- SmartPeak CLI binary or autoflowresearch/smartpeak-cli Docker container

## Outputs

- Per-injection wall-clock processing-time log (seconds per injection)
- throughput_benchmark.csv with columns: injection, processing time, computed automated-vs-manual speed-up ratio
- Exported featuredb and pivottable reports confirming the workflow ran to completion

## How to apply

Start from a self-contained example session — for SmartPeak, the FIAMS FullScan Unknowns dataset under src/examples/data/FIAMS_FullScan_Unknowns with its parameters.csv (acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20). Launch the SmartPeak CLI, optionally inside the autoflowresearch/smartpeak-cli Docker container, with --load-session pointing at the session directory. Run the full workflow command chain (LOAD_RAW_DATA, EXTRACT_SPECTRA_WINDOWS, MERGE_SPECTRA, PICK_MS1_FEATURES, SEARCH_ACCURATE_MASS, STORE_ANNOTATIONS, STORE_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, STORE_FEATURE_BACKGROUND_ESTIMATIONS, FILTER_FEATURES_BACKGROUND_INTERFERENCES, MERGE_FEATURES, MERGE_INJECTIONS, STORE_FEATURES_SAMPLE_GROUP) and instrument wall-clock time per injection via shell timing or the SmartPeak log. Export reports with --report featuredb pivottable to confirm the run completed end-to-end before trusting timings. Finally compile per-injection timings into a throughput table and compute the automated-vs-manual speed-up ratio, checking it against the paper's reported approximate 100-fold reduction.

## Related tools

- **SmartPeak** (Application whose automated workflow is timed per injection to produce the speed-up measurement) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line entry point used to load the session and run the workflow with instrumentation) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **OpenMS** (Underlying toolkit providing the peak-picking and feature-detection algorithms being timed)
- **Docker** (Container runtime used to launch the smartpeak-cli container for a reproducible timed run) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash
```

## Evaluation signals

- The full workflow command chain completes without errors for every injection in the sequence
- Per-injection timings are recorded for all injections and are consistent across the run (no anomalies or partial runs)
- The computed speed-up ratio is in the order of ~100-fold, consistent with the paper's reported reduction
- Report exports (featuredb, pivottable) are produced, confirming end-to-end processing rather than early termination
- throughput_benchmark.csv has the required columns (injection, processing time, speed-up ratio) and non-null values

## Limitations

- The ~100-fold figure is for data-processing time only, not for instrument acquisition time
- Estimated processing time shown in the SmartPeak GUI is a rough estimation that is updated regularly; the progress bar may not reflect remaining time
- Manual-baseline timing depends on analyst conventions and is not standardized in the paper, so the ratio is approximate

## Evidence

- [abstract] 100-fold reduction claim: "The application allows for an approximate 100-fold reduction in the data processing time compared to manual processing while enhancing quality and reproducibility of the results"
- [intro] Workflow automation scope: "The workflow automates all steps from peak detection and integration over calibration curve optimization, to quality control reporting."
- [readme] Example data location: [evidence span withheld — anchor status: external_doc]
- [readme] GUI time estimation caveat: [evidence span withheld — anchor status: external_doc]
- [methods] CLI invocation pattern: "docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash"
