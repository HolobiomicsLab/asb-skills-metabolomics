---
name: timing-statistics-aggregation
description: Use when you need to quantify and compare the runtime of a software tool
  across input categories (e.g., cluster types or module counts), requiring replicate
  timing and statistical aggregation.
when_to_use_negative:
- The goal is correctness or readability assessment of rendered diagrams rather than
  runtime measurement (use manual diagram assessment instead).
- Timing single monolithic runs without replicate categories — mean/std aggregation
  across categories requires replicate inputs per category.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Tool whose spaghetti-diagram rendering call is timed; also used for random
    BGC cluster generation to create benchmark inputs
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Provides time.perf_counter() for timing and pandas for mean/std aggregation
    of drawing times
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:28fd5fdb9d8a21e6c6e911633295c61cdfabd059f02d4138fab60b967fe3d4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# timing-statistics-aggregation

## Summary

Benchmark a tool's execution speed by wrapping its core call with Python's time.perf_counter(), then aggregate replicate timings into mean and standard deviation statistics per category using pandas. Applied here to RAIChU spaghetti-diagram rendering across PKS, NRPS, and hybrid BGC types.

## When to use

Use when you need to quantify and compare the runtime of a software tool across input categories (e.g., cluster types or module counts), requiring replicate timing and statistical aggregation.

## When NOT to use

- The goal is correctness or readability assessment of rendered diagrams rather than runtime measurement (use manual diagram assessment instead).
- Timing single monolithic runs without replicate categories — mean/std aggregation across categories requires replicate inputs per category.

## Inputs

- Randomly generated BGC specifications per category (5 module types x module counts 1-10 x 10 replicates)
- RAIChU cluster generation output feeding the timed drawing call
- Raw per-BGC timing records (seconds, float)

## Outputs

- Per-BGC timing rows (500 rows: 5 types x 10 counts x 10 BGCs)
- Aggregated mean and standard deviation of drawing seconds per module type and per module number
- drawing_time_results.csv timing table

## How to apply

Set up the tool environment exactly as documented — for RAIChU, a conda environment with Python 3.9.12 and 'pip install raichu'. Enumerate the categories to compare (here: 5 module types — NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS — crossed with 1–10 module counts) and generate replicate inputs (10 randomly generated BGCs per combination, 500 total) using RAIChU's cluster generation functionality. Wrap the expensive call (spaghetti-diagram rendering) with time.perf_counter() calls placed immediately before and after the drawing call so only the target operation is timed. Collect all durations into pandas and compute mean and standard deviation of drawing seconds per module type and per module number, writing the aggregated table to drawing_time_results.csv. Judge correctness by checking all 500 rows are present with finite, non-negative durations and that the qualitative speed ordering matches the reference (PKS fastest at 2.73 s/cluster).

## Related tools

- **RAIChU** (Tool whose spaghetti-diagram rendering call is timed; also used for random BGC cluster generation to create benchmark inputs) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Provides time.perf_counter() for timing and pandas for mean/std aggregation of drawing times)

## Examples

```
import time; start = time.perf_counter(); cluster.draw_spaghetti_diagram(); duration = time.perf_counter() - start
```

## Evaluation signals

- All 500 timing rows (5 module types x 10 module counts x 10 BGCs) are present in the results table.
- All recorded durations are non-negative and finite.
- Qualitative speed ordering matches the paper: PKS clusters drawn fastest (reported average 2.73 s per PKS cluster), NRPS and NRPS/PKS hybrid slower.
- Aggregated statistics include both mean and standard deviation per module type and per module number.
- Timing brackets only the drawing call (time.perf_counter() immediately around the RAIChU drawing call), excluding BGC generation overhead.

## Limitations

- RAIChU can only be installed via pip on Python 3.9.12 in the documented setup; newer Python versions require the biopython_independent branch.
- Randomly generated clusters may include programmatic shortcuts for some reactions (e.g., cationic cascade reactions in terpenes), so timings reflect simplified representations for those cases.
- Timing results are machine- and environment-dependent; only the qualitative speed ordering is a robust reproduction criterion, not absolute second values.

## Evidence

- [results] RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster: "RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster"
- [results] randomly generating spaghetti diagrams for 1-10 module BGCs, generating 10 BGCs per module number (1-10) and module type: "randomly generating spaghetti diagrams for 1–10 module BGCs, generating 10 BGCs per module number (1–10) and module type"
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs"
