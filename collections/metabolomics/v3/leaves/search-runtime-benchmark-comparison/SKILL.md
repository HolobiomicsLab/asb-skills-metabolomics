---
name: search-runtime-benchmark-comparison
description: Use when you need to quantify how long a spectral-vs-molecular-database
  search (exact or variable mode) takes at atlas scale and compare that cost to a
  baseline tool.
when_to_use_negative:
- Input spectra are already small enough that a legacy exact-search tool such as Dereplicator+
  would finish in reasonable time — the benchmark's value comes from atlas-scale (hundreds
  of millions of scans × tens of millions of molecules) comparisons.
- The analysis goal is variant discovery rather than runtime quantification — then
  use VInSMoC in variable mode directly instead of running a benchmark.
- You lack access to the npanalysis.org server or a comparable VInSMoC deployment;
  the runtime numbers are specific to that implementation and hardware configuration.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: Tool whose exact-mode, atlas-scale search runtime is measured; also the platform
    whose minimum-score and peak-preprocessing parameters are held at defaults during
    the benchmark.
  repo: https://run.npanalysis.org
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosting environment where the VInSMoC task is submitted, monitored through
    pending/running states, and from which runtime and result counts are recorded.
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:341d5cbc8593f591fbd9bfa202e88007d9d1f5aa2ab078e6daf4ad319ca2ad8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# search-runtime-benchmark-comparison

## Summary

Benchmarks the wall-clock runtime and identification yield of an exact-mode, atlas-scale mass spectral database search (VInSMoC against GNPS spectra vs. PubChem molecules) and contextualizes it against an estimate for a legacy tool such as Dereplicator+.

## When to use

Use when you need to quantify how long a spectral-vs-molecular-database search (exact or variable mode) takes at atlas scale and compare that cost to a baseline tool.

## When NOT to use

- Input spectra are already small enough that a legacy exact-search tool such as Dereplicator+ would finish in reasonable time — the benchmark's value comes from atlas-scale (hundreds of millions of scans × tens of millions of molecules) comparisons.
- The analysis goal is variant discovery rather than runtime quantification — then use VInSMoC in variable mode directly instead of running a benchmark.
- You lack access to the npanalysis.org server or a comparable VInSMoC deployment; the runtime numbers are specific to that implementation and hardware configuration.

## Inputs

- GNPS mass spectral query dataset (MGF, mzXML, or mzML spectra files, e.g. from MassIVE accession MSV000086287 for a smoke test)
- PubChem (and optionally COCONUT) candidate molecule chemical database
- VInSMoC task configuration: exact search mode, ion mode (positive/negative), precursor and product ion tolerances, minimum score 30

## Outputs

- Total wall-clock runtime (hours) and thread count for the search (e.g. 305.12 hours on 128 threads)
- Count of computed molecule–spectrum matches (e.g. 6,282,126,171,302 for 483,537,388 scans × 87,032,328 molecules)
- Count of unique known-molecule identifications above the score threshold (≈43,598 at minimum score 30)
- Estimated baseline runtime for a comparison tool (e.g. ~30 years for Dereplicator+)

## How to apply

Configure a VInSMoC task on the npanalysis.org server with search mode set to exact, ion mode matching the spectra, and default precursor/product ion tolerances and peak preprocessing (Peak Merge Threshold with Minimum Peak Count, and Peaks Per Window most intense peaks per Peak Filtering Window). Launch the search over the full GNPS query set against the PubChem/COCONUT candidate database, record the total wall-clock runtime and the thread count used, and export best-hit-per-scan results scored above the minimum score threshold of 30. For the benchmark in the article, 6,282,126,171,302 molecule–spectrum matches (483,537,388 GNPS scans × 87,032,328 PubChem molecules) completed in 305.12 hours on 128 threads. Compare this measured runtime to an estimated runtime for a slower baseline such as Dereplicator+ (~30 years for the same search) to quantify the scalability gain.

## Related tools

- **VInSMoC** (Tool whose exact-mode, atlas-scale search runtime is measured; also the platform whose minimum-score and peak-preprocessing parameters are held at defaults during the benchmark.) — https://run.npanalysis.org
- **npanalysis.org server (run.npanalysis.org)** (Hosting environment where the VInSMoC task is submitted, monitored through pending/running states, and from which runtime and result counts are recorded.) — https://run.npanalysis.org

## Evaluation signals

- The recorded runtime should be comparable to the reported 305.12 hours (12.7 days) on 128 threads for the same GNPS-vs-PubChem exact-mode search.
- The match count should equal the product of input scans and candidate molecules — reported as 6,282,126,171,302 matches from 483,537,388 scans × 87,032,328 molecules.
- The exported best-hit results should yield approximately 43,598 unique known-molecule identifications at minimum score 30.
- The estimated baseline runtime for Dereplicator+ should be on the order of ~30 years for the same search, confirming the measured speedup.
- For a small smoke-test run (e.g. the MSV000086287 spectrum file), the task should transition from pending to finished rapidly, validating the task setup before launching the full-scale benchmark.

## Limitations

- The reported runtime is specific to the npanalysis.org deployment (128 threads); results will vary with different hardware or thread counts.
- The comparison to Dereplicator+ is an estimate (~30 years), not a directly measured run, so the speedup figure is approximate.
- Exact-mode searches only match molecules whose masses match the precursor m/z; the runtime benchmark does not capture the additional cost of variable mode, where molecule variants must explain precursor mass differences.

## Evidence
