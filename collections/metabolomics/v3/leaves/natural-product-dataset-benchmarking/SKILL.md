---
name: natural-product-dataset-benchmarking
description: Use when comparing GPU vs CPU runtimes of the CSCCP dynamic-programming
  solver across natural-product mixture datasets, or when reproducing the reported
  ~20.72 s vs 324.16 s runtime at D=5 mass precision.
when_to_use_negative:
- The goal is structure elucidation accuracy of unknown mixtures rather than runtime
  benchmarking — use the full G.A.M.E. prediction workflow instead
- Mass precision other than D=1–5 is required; the DP algorithm's runtime grows by
  orders of magnitude as decimal digits increase, so D far above 5 is impractical
  for the CPU reference
- No CUDA-capable GPU is available — G.A.M.E.'s acceleration depends on GPU parallelization
  over w values
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver (csccp-solver-cli) whose per-dataset wall-clock
    runtime is being benchmarked
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit enabling the parallel DP implementation being timed
- name: PyCuda 2017.1
  role: Python binding that lets the Python 2.7 harness invoke the CUDA kernels during
    timed runs
- name: NVIDIA Tesla K40c
  role: Reference GPU hardware (compute capability 3.5, 15 streaming multiprocessors
    of 192 cores each) on which timings are measured
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:2a18ff7d7c3c1c00cf9b90d695ff8c53b84f731a957d1310e464cecce2a3e0c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA 7.5
  - G.A.M.E.
  - NVIDIA Tesla K40c
  - PyCuda 2017.1
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# natural-product-dataset-benchmarking

## Summary

Benchmark the GPU-accelerated DP solver (G.A.M.E.) against the CPU DP reference on four natural-product testing datasets at five decimal digits of mass precision, measuring wall-clock runtimes and computing speed-up factors into a CSV table.

## When to use

Use when comparing GPU vs CPU runtimes of the CSCCP dynamic-programming solver across natural-product mixture datasets, or when reproducing the reported ~20.72 s vs 324.16 s runtime at D=5 mass precision.

## When NOT to use

- The goal is structure elucidation accuracy of unknown mixtures rather than runtime benchmarking — use the full G.A.M.E. prediction workflow instead
- Mass precision other than D=1–5 is required; the DP algorithm's runtime grows by orders of magnitude as decimal digits increase, so D far above 5 is impractical for the CPU reference
- No CUDA-capable GPU is available — G.A.M.E.'s acceleration depends on GPU parallelization over w values

## Inputs

- Seed scaffold files (*.cIdx) from GAME/examples/core_index
- Dataset scaffold index lists from GAME/examples/Datasets/scaffolds for the four testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.)
- Targeted molecular weight values from GAME/examples/MW/ (parameter -m)
- NVIDIA Tesla K40c GPU (compute capability 3.5) with CUDA 7.5 and PyCuda 2017.1 on Python 2.7

## Outputs

- runtime_benchmark.csv with exactly four dataset rows: dataset name, G.A.M.E. runtime (s), CPU DP reference runtime (s), speed-up factor
- Per-execution wall-clock timings averaged over random samplings of 20 cases per dataset

## How to apply

Clone github.com/CMDM-Lab/GAME (also archived at Zenodo doi 10.5281/zenodo.237579) which packages seed scaffold files (*.cIdx) under GAME/examples/core_index, dataset scaffold indices under GAME/examples/Datasets/scaffolds, and targeted molecular weights under GAME/examples/MW/. For each of the four testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.), run csccp-solver-cli over the dataset's seed scaffolds with the DP solver (-v idp, -v cc), setting --dec 5 (D=5), -r 3, configuration index -c taken from the scaffold file's line 3, and the mass value (-m) taken from GAME/examples/MW/. Measure wall-clock runtime of each G.A.M.E. execution, averaging over random samplings of cases within each dataset (e.g. 10 samplings of 20 cases per dataset) per the paper's protocol. Take the CPU DP reference times from the article text (average 324.16 s at D=5) rather than re-executing, and compute speed-up as CPU DP time divided by G.A.M.E. time. Write one CSV (runtime_benchmark.csv) with exactly four dataset rows containing dataset name, G.A.M.E. runtime, CPU DP reference runtime, and speed-up factor.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver (csccp-solver-cli) whose per-dataset wall-clock runtime is being benchmarked) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit enabling the parallel DP implementation being timed)
- **PyCuda 2017.1** (Python binding that lets the Python 2.7 harness invoke the CUDA kernels during timed runs)
- **NVIDIA Tesla K40c** (Reference GPU hardware (compute capability 3.5, 15 streaming multiprocessors of 192 cores each) on which timings are measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- runtime_benchmark.csv has row_count_equals 4 (one row per testing dataset)
- Each G.A.M.E. runtime completes within a bounded evaluation window (well under 24 hours), consistent with the reported ~20.72 s average at D=5
- Speed-up factor equals CPU DP reference time divided by G.A.M.E. time, on the order of tens (article reports an average acceleration factor of 33 in 99% of the cases)
- Averaging protocol matches the paper: ~10 samplings of 20 cases per dataset, with datasets limited to the four natural products
- csccp-solver-cli output confirms the expected csccp info block (mass_peak, configuration, top-r ranked structures) for each timed run

## Limitations

- CPU DP reference times are taken from the article text (average 324.16 s at D=5) rather than re-executed, so speed-up factors depend on historical hardware
- DP runtime grows by orders of magnitude as mass decimal digits increase — roughly tenfold per additional digit — limiting practical D to 1–5
- Benchmark is tied to the NPDB-derived scaffold database (83,242 scaffolds from 243,130 natural products; 26,641 suitable with wmax = 500, 51,562 total configurations) and specific GPU hardware (Tesla K40c); results may not transfer to other scaffold sets or GPUs

## Evidence

- [abstract] CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [methods] Python 2.7 and CUDA 7.5 with PyCuda 2017.1 on Tesla K40c: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] csccp-solver-cli invocation from README: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [intro] Repository availability statement: "All datasets and source code of G.A.M.E. are available on GitHub at https://github.com/CMDM-Lab/GAME."
- [discussion] Acceleration factor reported in discussion: "an average acceleration factor of 33 in 99% of the cases"
