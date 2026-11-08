---
name: runtime-benchmark-comparison
description: Use when two competing parameter-optimization or data-processing tools
  must be compared on identical raw-data inputs (e.g. MTBLS157 mzML files) with different
  CPU allocations, and a normalized wall-clock speed ratio is the reported outcome.
when_to_use_negative:
- The goal is to compare parameter-estimate accuracy or returned feature tables between
  AutoTuner and IPO rather than runtime (accuracy comparisons found IPO could under-select
  peak-width parameters, a separate evaluation).
- The input is already a processed feature table — both tools optimize parameters
  from raw mzML/mzXML/CDF data, so no runtime optimization comparison is meaningful.
- 'You expect AutoTuner to benefit from more CPUs: it is not parallelized, so scaling
  CPU count does not change its side of the benchmark.'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: 'Benchmark subject: runs its single-step raw-data parameter estimation on
    1 CPU and its wall-clock runtime is recorded'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: IPO
  role: 'Comparison baseline: iterative isotopologue parameter optimization run on
    8 CPUs, whose wall-clock runtime defines the other side of the speed ratio'
- name: R
  role: Runtime environment used to execute both AutoTuner (R >= 3.6 via BioConductor)
    and IPO and to record runtimes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:d6f461ecac095037cb3cbbcc7acfd8bbb30f02dc79ccf7c7300b377160962291
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# runtime-benchmark-comparison

## Summary

Benchmarks wall-clock runtimes of two parameter-optimization tools (AutoTuner on 1 CPU vs IPO on 8 CPUs) on the same untargeted metabolomics mzML dataset and reports a CPU-count-adjusted speed ratio, validating the paper's claimed ~100-1000X improvement.

## When to use

Use when two competing parameter-optimization or data-processing tools must be compared on identical raw-data inputs (e.g. MTBLS157 mzML files) with different CPU allocations, and a normalized wall-clock speed ratio is the reported outcome.

## When NOT to use

- The goal is to compare parameter-estimate accuracy or returned feature tables between AutoTuner and IPO rather than runtime (accuracy comparisons found IPO could under-select peak-width parameters, a separate evaluation).
- The input is already a processed feature table — both tools optimize parameters from raw mzML/mzXML/CDF data, so no runtime optimization comparison is meaningful.
- You expect AutoTuner to benefit from more CPUs: it is not parallelized, so scaling CPU count does not change its side of the benchmark.

## Inputs

- MTBLS157 bacterial culture dataset raw mzML files from MetaboLights
- Two-col spreadsheet mapping sample names to experimental factors (required by AutoTuner alongside the raw data)

## Outputs

- Table of recorded wall-clock runtimes for AutoTuner (1 CPU) and IPO (8 CPUs)
- CPU-count-adjusted speed ratio (IPO time * 8 vs AutoTuner time * 1)

## How to apply

Download the MTBLS157 bacterial-culture mzML raw files from MetaboLights and run AutoTuner's single-step parameter-optimization workflow on a single CPU, recording wall-clock time. Run IPO's iterative parameter optimization on the same mzML files with 8 CPUs and record its wall-clock time. Compute the CPU-adjusted ratio as IPO time multiplied by its 8 CPUs versus AutoTuner time multiplied by 1 CPU, since AutoTuner is not parallelized. Compare the resulting ratio against the paper's reported hundreds-to-thousands-of-times speedup (Table 3) and write the runtimes and adjusted ratio to a results table. The rationale is that AutoTuner performs statistical inference on raw data in one step while IPO iteratively checks parameter combinations, which is why the paper reports speedups 'going from days to minutes'.

## Related tools

- **AutoTuner** (Benchmark subject: runs its single-step raw-data parameter estimation on 1 CPU and its wall-clock runtime is recorded) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **IPO** (Comparison baseline: iterative isotopologue parameter optimization run on 8 CPUs, whose wall-clock runtime defines the other side of the speed ratio)
- **R** (Runtime environment used to execute both AutoTuner (R >= 3.6 via BioConductor) and IPO and to record runtimes)

## Examples

```
Rscript -e 'BiocManager::install(version="devel"); BiocManager::install("Autotuner")'
```

## Evaluation signals

- The CPU-adjusted ratio (AutoTuner time * 1 vs IPO time * 8) falls within the reported ~100-1000X range, matching Table 3 of the paper.
- Both tools were run on the identical MTBLS157 mzML file set, so runtimes are directly comparable.
- Wall-clock times are recorded per run and reproducible on a single machine, with the CPU multiplier applied only to IPO.
- AutoTuner completes in minutes while IPO completes on the scale of hours-to-days, consistent with the 'days to minutes' claim.
- The output table lists both raw runtimes and the adjusted ratio, not just one number.

## Limitations

- AutoTuner is not parallelized; the paper notes it 'could be parallelized to reduce computation time by a factor of the total number of CPUs used', so multi-CPU comparisons favor IPO in raw CPU allocation.
- The measured speedup reflects AutoTuner's 'divide-and-conquer' approach, which forgoes comparing EIC peaks across samples for retention-time correction — the runtime gain trades away that cross-sample step.
- AutoTuner's sliding-window analysis requires user input in its first portion, which may add non-computational time to the recorded wall clock.
- Results are specific to datasets like MTBLS157 (qTOF/orbitrap/FT-ICR data); the paper notes no universal parameter set exists for all datasets.

## Evidence

- [abstract] claimed speedup magnitude: "it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes)."
- [intro] motivation for runtime benchmarking vs HPC-dependent baselines: "Currently, these parameter selection algorithms depend on high performance computing resources."
- [discussion] why AutoTuner runs on 1 CPU in the comparison: "First, AutoTuner could be parallelized to reduce computation time by a factor of the total number of CPUs used."
- [intro] rationale for the speed gap (single-step vs iterative): "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [readme] input format and minimum-sample requirements for the benchmark run: [evidence span withheld — anchor status: external_doc]
