---
name: precursor-ion-tolerance-sweep-benchmarking
description: Use when you need to characterize how runtime of a mass spectral search
  tool (e.g. VInSMoC) scales as precursor ion tolerance increases, or to compare exact
  vs. variable search cost head-to-head.
when_to_use_negative:
- Input spectra have already been pre-processed off-server (Peak Merge Threshold /
  Peak Filtering) and the goal is scoring accuracy rather than runtime benchmarking
- The goal is molecule identification output itself rather than measuring runtime
  scaling across precursor tolerance thresholds
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Search engine whose runtime is benchmarked in exact and variable search modes
    across precursor ion tolerance values
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web server where VInSMoC tasks are created, submitted, and monitored through
    pending/running/finished states
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:eedf0455f64ac5b2a0d5f189cd7dd24baed27b96c474bf16e430b42178f537bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# precursor-ion-tolerance-sweep-benchmarking

## Summary

Benchmarks VInSMoC runtime in exact and variable search modes by sweeping precursor ion tolerance values over one mass spectral search task per tolerance, comparing against other search engines such as Dereplicator+ on a fixed spectrum/database pair.

## When to use

Use when you need to characterize how runtime of a mass spectral search tool (e.g. VInSMoC) scales as precursor ion tolerance increases, or to compare exact vs. variable search cost head-to-head.

## When NOT to use

- Input spectra have already been pre-processed off-server (Peak Merge Threshold / Peak Filtering) and the goal is scoring accuracy rather than runtime benchmarking
- The goal is molecule identification output itself rather than measuring runtime scaling across precursor tolerance thresholds

## Inputs

- Query mass spectrum file in MGF, mzXML, or mzML format (e.g. a file from MSV000086287)
- Chemical database such as NPAtlas associated to the project
- Series of precursor ion tolerance threshold values to sweep

## Outputs

- runtime_benchmark.csv with wall-clock runtime per (search mode, precursor ion tolerance) combination
- Finished-state task pages reporting the best hit per input scan exceeding the minimum score reporting threshold

## How to apply

Upload a query mass spectrum file (MGF, mzXML, or mzML, e.g. from MSV000086287) and a chemical database such as NPAtlas to a project on run.npanalysis.org. Create VInSMoC tasks with Peptide fragmentation mode and the ion mode matching the input spectrum (positive), leaving product ion tolerance and preprocessing (Peak Merge Threshold, Peaks Per Window, Peak Filtering Window) at defaults. For each of a series of monotonically increasing precursor ion tolerance values, submit one task in exact search mode and one in variable search mode, then record wall-clock runtime from submission until the task leaves pending/running and reaches the finished state into a runtime table (e.g. runtime_benchmark.csv). Compare recorded runtimes between exact and variable modes across the tolerance sweep and, if replicating the paper's comparison, against other engines such as Dereplicator+ measured at the same thresholds.

## Related tools

- **VInSMoC** (Search engine whose runtime is benchmarked in exact and variable search modes across precursor ion tolerance values) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web server where VInSMoC tasks are created, submitted, and monitored through pending/running/finished states) — https://run.npanalysis.org

## Evaluation signals

- Each task in the sweep reaches the finished state (not error) after pending/running
- Task results page reports the best hit to each input scan that exceeded the minimum score reporting threshold
- Runtimes are recorded for every (search mode, precursor tolerance) combination, not just a subset
- Comparison holds at every measured precursor ion threshold when pitted against engines like Dereplicator+

## Limitations

- Large input sizes can cause tasks to spend substantial time in the 'Running' state before finishing
- Variable mode performs more complicated scoring than exact mode, so runtime behavior differs between modes and must be measured separately at each tolerance
- If the spectrum input file is malformed relative to its declared extension, the data point enters an error state and must be re-uploaded

## Evidence

- [results] VInSMoC in variable search with metabolite fragmentation mode is faster than Dereplicator+ at every measured precursor ion threshold: [evidence span withheld — anchor status: not_found]
