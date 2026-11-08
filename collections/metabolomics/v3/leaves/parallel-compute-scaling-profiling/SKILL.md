---
name: parallel-compute-scaling-profiling
description: Use when comparing computational cost or parallel-scaling behavior of
  competing normalization/analysis methods (e.g., Metanorm's QC-RLSC/QC-RSC/rLOESS/rGAM/tGAM
  vs TIGER or SERFF) on the same data sets.
when_to_use_negative:
- The goal is assessing normalization quality (drift/batch-effect removal, replicate
  concordance) rather than computational performance — use accuracy-based benchmarking
  instead
- Methods expose no core-count control (e.g., SERFF is only reachable through its
  Shiny web interface, so core scaling cannot be profiled)
- The data set is too small to amortize parallelization overhead, which the reference
  shows produces tapering/declining scaling curves on BioHEART
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Metanorm
  role: R package supplying the normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM,
    tGAM) to be timed, exploiting pleasingly parallel processing across cores
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Installable implementation in which each method is invoked with 1–6 compute
    cores and runtimes recorded
  repo: https://github.com/UGent-LIMET/Metanorm
- name: TIGER
  role: External multivariate normalization baseline run through its R package under
    the same core-scaling scenarios
- name: SERFF
  role: External multivariate normalization baseline run through its Shiny web interface;
    core scaling not applicable, timed once
- name: R
  role: Runtime environment (>= 4.4.0) in which the benchmark scripts, timing, and
    throughput calculations are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:f2a8876584a9135989a8a36b85b575c996e05af38468a7f7fea64a7879ae3ae6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Parallel-compute scaling profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the computational runtime and parallel-scaling behavior of metabolomics normalization methods by timing each method on fixed targeted and untargeted data sets while varying CPU core count (1–6), and expressing throughput as normalized compounds per minute to expose which methods scale and which saturate.

## When to use

Use when comparing computational cost or parallel-scaling behavior of competing normalization/analysis methods (e.g., Metanorm's QC-RLSC/QC-RSC/rLOESS/rGAM/tGAM vs TIGER or SERFF) on the same data sets.

## When NOT to use

- The goal is assessing normalization quality (drift/batch-effect removal, replicate concordance) rather than computational performance — use accuracy-based benchmarking instead
- Methods expose no core-count control (e.g., SERFF is only reachable through its Shiny web interface, so core scaling cannot be profiled)
- The data set is too small to amortize parallelization overhead, which the reference shows produces tapering/declining scaling curves on BioHEART

## Inputs

- Targeted metabolomics data set (e.g., BioHEART: 53 compounds x 1361 sample runs, 15 batches) as a numerical intensity matrix
- Untargeted metabolomics data set (e.g., FAME: 8913 features x 618 runs, 9 batches) as a numerical intensity matrix
- Per-run batch labels and QC/sample-type vectors
- Metanorm R package, TIGER R package, and SERFF Shiny interface
- Compute environment with configurable core counts (1–6)

## Outputs

- Raw runtimes (seconds/minutes) per method per data set per core count
- Normalized-throughput table: method x dataset x core count in compounds per minute
- Compounds-per-minute versus core-count scaling curves per method and dataset

## How to apply

Obtain both a small targeted data set (e.g., BioHEART: 53 compounds, 1361 sample runs, 15 batches) and a large untargeted data set (e.g., FAME: 8913 features, 618 runs, 9 batches), and install the software under test (Metanorm R package via pak, requires R >= 4.4.0; TIGER via its R package; SERFF via its Shiny web interface). Run each method under identical scenarios with 1 through 6 compute cores, timing each analysis with system.time or equivalent; insert a 2 min break between consecutive method analyses to prevent CPU throttling from confounding the timings. Use each tool's defaults except where the reference workflow specifies (e.g., rGAM and tGAM run with the QCcheck argument active); SERFF's web interface makes core scaling inapplicable, so record it once. Convert every raw runtime to normalized compounds per minute, compile a method-by-dataset-by-core-count table, and plot compounds-per-minute scaling curves versus number of cores. Interpret against reference anchors: tGAM ~136 s and TIGER 64 s on BioHEART, tGAM ~15 min and TIGER ~115 min on FAME, with QC-RSC fastest overall, fastest methods showing tapering/declining scaling on the small BioHEART set but steady improvement with more cores on FAME.

## Related tools

- **Metanorm** (R package supplying the normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) to be timed, exploiting pleasingly parallel processing across cores) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Installable implementation in which each method is invoked with 1–6 compute cores and runtimes recorded) — https://github.com/UGent-LIMET/Metanorm
- **TIGER** (External multivariate normalization baseline run through its R package under the same core-scaling scenarios)
- **SERFF** (External multivariate normalization baseline run through its Shiny web interface; core scaling not applicable, timed once)
- **R** (Runtime environment (>= 4.4.0) in which the benchmark scripts, timing, and throughput calculations are executed)

## Examples

```
system.time(normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch))
```

## Evaluation signals

- Reproduced timings fall near reported anchors: tGAM ~136 s and TIGER 64 s on BioHEART (53 compounds); tGAM ~15 min and TIGER ~115 min on FAME (8913 features)
- QC-RSC ranks fastest overall across both data sets
- Scaling curves for the fastest methods taper or decline with added cores on the small BioHEART set, while FAME shows steady throughput improvement up to 6 cores
- Throughput table is complete for every method x dataset x core-count cell (SERFF exempt from core scaling)
- CPU-throttling artifacts are controlled (2 min break between consecutive method analyses) so runtimes are reproducible across repetitions

## Limitations

- Runtimes are environment-dependent; the 2 min break between analyses is required to prevent CPU throttling, so results may not transfer across hardware
- Small data sets saturate quickly: adding cores yields tapering or declining compounds-per-minute curves for the fastest methods
- Multivariate methods benchmarked via web interfaces (SERFF) cannot be core-scaled; the article also notes benchmarking neutrality requires care to avoid favoring any method

## Evidence

- [results] Reference timings on BioHEART: [evidence span withheld — anchor status: not_found]
- [readme] Version prerequisite for the benchmark environment: [evidence span withheld — anchor status: external_doc]
- [readme] Core function whose runtime is measured: [evidence span withheld — anchor status: external_doc]
