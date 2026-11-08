---
name: search-runtime-profiling
description: Use when benchmarking VInSMoC's runtime in exact and variable search
  modes against other mass spectral search engines as precursor ion tolerance increases.
when_to_use_negative:
- The goal is spectral identification itself (score-based matching) rather than runtime
  benchmarking — use the VInSMoC identification workflow instead.
- Input spectra require custom preprocessing or non-default peak merging/filtering
  settings, since the benchmark protocol uses default preprocessing parameters.
- The search target is a database of small molecules not supported by the server workflow
  (e.g. a custom peptide-only search outside NPAtlas/PubChem/COCONUT scope described
  here).
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Mass spectral search tool run in exact and variable modes across a precursor
    ion tolerance sweep to profile runtime
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosted platform used to create projects, submit VInSMoC tasks, and monitor
    pending/running/finished states for runtime recording
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:8b0cfa70e75b212fe366399f1c5233c2f0edd1b002cf9a62cf00122610933c61
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# search-runtime-profiling

## Summary

Profiles the wall-clock runtime of VInSMoC database searches (exact vs. variable mode) by sweeping precursor ion tolerances on the npanalysis.org server, enabling runtime comparison against other mass spectral search engines such as Dereplicator+.

## When to use

Use when benchmarking VInSMoC's runtime in exact and variable search modes against other mass spectral search engines as precursor ion tolerance increases.

## When NOT to use

- The goal is spectral identification itself (score-based matching) rather than runtime benchmarking — use the VInSMoC identification workflow instead.
- Input spectra require custom preprocessing or non-default peak merging/filtering settings, since the benchmark protocol uses default preprocessing parameters.
- The search target is a database of small molecules not supported by the server workflow (e.g. a custom peptide-only search outside NPAtlas/PubChem/COCONUT scope described here).

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format (e.g. from MSV000086287)
- Chemical database associated to the project (e.g. NPAtlas)
- Series of precursor ion tolerance values for the sweep

## Outputs

- runtime_benchmark.csv with wall-clock runtime per (search mode, precursor tolerance) combination
- Comparison of VInSMoC exact/variable search runtimes across the tolerance sweep

## How to apply

Upload mass spectral inputs (MGF, mzXML, or mzML, e.g. from MSV000086287) and associate the NPAtlas chemical database to a project on run.npanalysis.org. Create a VInSMoC task with Peptide fragmentation mode, ion mode matching the input spectra (positive), and product ion tolerance/preprocessing left at defaults. Run VInSMoC in exact search mode, sweeping the precursor ion tolerance over a series of increasing values with one task per tolerance, then repeat the sweep with search mode set to variable. Record the wall-clock runtime from submission to the finished state for every (search mode, precursor tolerance) combination into runtime_benchmark.csv. Confirm each task reaches the finished state and its results page reports the best hit per input scan exceeding the minimum score threshold before comparing runtimes.

## Related tools

- **VInSMoC** (Mass spectral search tool run in exact and variable modes across a precursor ion tolerance sweep to profile runtime) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosted platform used to create projects, submit VInSMoC tasks, and monitor pending/running/finished states for runtime recording) — https://run.npanalysis.org

## Examples

```
VInSMoC task via run.npanalysis.org: create task with Search mode=variable, Fragmentation mode=Peptide, Ion mode=positive, sweep Precursor ion tolerance, record runtime in runtime_benchmark.csv
```

## Evaluation signals

- Every submitted VInSMoC task reaches the finished state (no error or stalled pending/running states).
- Each task's results page reports the best hit per input scan that exceeded the minimum score reporting threshold.
- runtime_benchmark.csv contains one runtime row per (search mode, precursor tolerance) combination in the sweep.
- VInSMoC in variable search with metabolite fragmentation mode completes faster than Dereplicator+ at every measured precursor ion threshold despite performing more complicated scoring.
- Runtimes for VInSMoC peptide-fragmentation exact and variable settings show a consistent trend across increasing tolerances.

## Limitations

- Runtimes depend on the input data size and chemical database; the described example uses a spectrum file from MSV000086287 and the NPAtlas database.
- Tasks may remain in the pending state while execution is scheduled, and small examples finish almost immediately whereas large inputs can spend substantial time in the running state.
- The protocol uses default preprocessing (peak merging and peak filtering) parameters; changing these may affect comparability of runtimes.
- The comparison is specific to Dereplicator+/VarQuest on the GNPSLIBvNPAtlas dataset; conclusions may not generalize to other search engines or datasets.

## Evidence

- [results] VInSMoC in variable search with metabolite fragmentation mode is faster than Dereplicator+ at every measured precursor ion threshold: [evidence span withheld — anchor status: not_found]
