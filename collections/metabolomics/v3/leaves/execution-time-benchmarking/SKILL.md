---
name: execution-time-benchmarking
description: Use when you need to quantify how fast a tool (e.g. RAIChU drawing biosynthetic
  pathway diagrams) executes across input categories, and produce a timing table supporting
  performance comparisons.
when_to_use_negative:
- The goal is correctness or readability assessment of spaghetti diagrams rather than
  runtime measurement — that requires manual structural scoring, not timing
- The input clusters are real antiSMASH predictions of MIBiG BGCs rather than randomly
  generated modular clusters with controlled module counts
- The tool under test cannot be invoked programmatically inside a Python timing loop
  (RAIChU is pip-installable precisely to enable this)
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RAIChU
  role: 'Tool under benchmark: its cluster generation functionality creates randomized
    test BGCs and its drawing routine (spaghetti diagrams) is the timed operation'
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: 'Benchmark harness: time.perf_counter() for timing, random generation loop,
    and pandas aggregation of the 500 timing rows'
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:5ac05a4a4136fcf69a8377e75556bbeae14f0864750a5d998f2948d210682e13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# execution-time-benchmarking

## Summary

Benchmark the runtime of a scientific software tool (here, RAIChU's spaghetti-diagram rendering of PKS/NRPS BGCs) by timing repeated, randomized runs with time.perf_counter() and aggregating mean and standard deviation per category. Used to establish speed orderings and per-unit costs of automated visualization.

## When to use

Use when you need to quantify how fast a tool (e.g. RAIChU drawing biosynthetic pathway diagrams) executes across input categories, and produce a timing table supporting performance comparisons.

## When NOT to use

- The goal is correctness or readability assessment of spaghetti diagrams rather than runtime measurement — that requires manual structural scoring, not timing
- The input clusters are real antiSMASH predictions of MIBiG BGCs rather than randomly generated modular clusters with controlled module counts
- The tool under test cannot be invoked programmatically inside a Python timing loop (RAIChU is pip-installable precisely to enable this)

## Inputs

- Randomly generated PKS/NRPS BGC clusters (1–10 modules) from RAIChU's cluster generation functionality
- 5 module-type categories: NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS
- Pinned Python 3.9.12 conda environment with raichu installed via pip

## Outputs

- Per-run wall-clock drawing times in seconds (measured with time.perf_counter())
- Aggregated timing table (mean and standard deviation per module type and per module number) written to drawing_time_results.csv
- Qualitative speed ordering of cluster types (e.g. PKS fastest at 2.73 s per cluster vs 9.62 s NRPS and 5.21 s hybrid)

## How to apply

Set up the tool in a pinned environment (conda create -n raichu python==3.9.12, then pip install raichu). Define representative input categories — for RAIChU, the 5 module types (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS) crossed with module counts 1–10 — and randomly generate multiple replicates per category (10 BGCs each, 500 total) using RAIChU's cluster generation functionality. For each replicate, wrap the drawing/rendering call in Python's time.perf_counter() to measure wall-clock seconds, then aggregate with pandas computing mean and standard deviation per module type (and per module number) and write the table to drawing_time_results.csv. Judge success by confirming the expected qualitative speed ordering (PKS fastest at a reported average of 2.73 s per cluster; hybrids and NRPS slower) and by verifying row count and data sanity (all rows present, durations finite and non-negative).

## Related tools

- **RAIChU** (Tool under benchmark: its cluster generation functionality creates randomized test BGCs and its drawing routine (spaghetti diagrams) is the timed operation) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Benchmark harness: time.perf_counter() for timing, random generation loop, and pandas aggregation of the 500 timing rows) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- All 500 expected timing rows are present (5 module types x 10 module counts x 10 BGCs)
- Every recorded duration is finite and non-negative
- The mean-speed ordering reproduces the paper's qualitative result: PKS clusters drawn fastest (reported average 2.73 s per PKS cluster)
- Mean and standard deviation are computed per module type and per module number via pandas
- Output file drawing_time_results.csv is produced with the aggregated table

## Limitations

- Drawing speed depends on cluster type and module count; PKS clusters are systematically faster than NRPS or NRPS/PKS hybrids, so cross-type comparisons require the category breakdown
- Timing must use randomized BGCs with matched module-count distributions (10 BGCs per module number 1–10) or the comparison is biased
- RAIChU results assume Python 3.9.12; newer Python versions require the biopython_independent branch, so timings may not transfer across environments

## Evidence

- [results] RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster: "RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster"
- [results] speed assessment design: "randomly generating spaghetti diagrams for 1–10 module BGCs, generating 10 BGCs per module number (1–10) and module type"
- [methods] environment setup: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] pip installation: "Then, install raichu with pip: pip install raichu"
