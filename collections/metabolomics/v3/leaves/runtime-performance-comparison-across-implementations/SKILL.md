---
name: runtime-performance-comparison-across-implementations
description: Use when a paper claims GPU acceleration makes an NP-complete combinatorial
  solve (e.g. CSCCP structure elucidation at dec=5) practical, and you must reproduce
  the wall-clock runtime comparison versus the CPU DP baseline.
when_to_use_negative:
- Input masses require more than five decimal digits of precision — G.A.M.E. is validated
  only up to dec=5, and CPU DP runtimes grow by orders of magnitude with each additional
  decimal digit
- The goal is accuracy or ranking validation of elucidated structures (top-3 molecule
  SMILES/probabilities) rather than runtime comparison of implementations
- No CUDA-capable GPU or GPU software stack (CUDA/PyCuda) is available — the G.A.M.E.
  arm of the benchmark cannot be executed
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver (csccp-solver-cli) whose wall-clock runtime is measured
    at dec=5 for the four testing datasets
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit underpinning the G.A.M.E. parallelization being benchmarked
    against the CPU DP baseline
- name: PyCuda 2017.1
  role: Python binding used by G.A.M.E. to launch the CUDA kernels timed in the benchmark
- name: NVIDIA Tesla K40c
  role: Reference GPU hardware (compute capability 3.5, 15 SMs × 192 cores) on which
    benchmark timings are taken
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:a7e3abe8f046411c08b3f3da9fbb12d8c2526ba0e2c717d450f6fbd909c46111
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# runtime-performance-comparison-across-implementations

## Summary

Benchmarks a GPU-accelerated solver (G.A.M.E., the CUDA-based mixture elucidator) against the reported CPU dynamic programming (DP) runtimes at five decimal digits of mass precision (dec=5), producing a per-dataset runtime and speed-up table for the four natural-product testing datasets.

## When to use

Use when a paper claims GPU acceleration makes an NP-complete combinatorial solve (e.g. CSCCP structure elucidation at dec=5) practical, and you must reproduce the wall-clock runtime comparison versus the CPU DP baseline.

## When NOT to use

- Input masses require more than five decimal digits of precision — G.A.M.E. is validated only up to dec=5, and CPU DP runtimes grow by orders of magnitude with each additional decimal digit
- The goal is accuracy or ranking validation of elucidated structures (top-3 molecule SMILES/probabilities) rather than runtime comparison of implementations
- No CUDA-capable GPU or GPU software stack (CUDA/PyCuda) is available — the G.A.M.E. arm of the benchmark cannot be executed

## Inputs

- Seed scaffold files (*.cIdx) from GAME/examples/core_index, with per-dataset scaffold index lists in GAME/examples/Datasets/scaffolds
- Targeted molecular weights per dataset from GAME/examples/MW/ (passed via the -m parameter)
- GPU workstation meeting G.A.M.E. dependencies (CUDA 7.5, PyCuda 2017.1, Python 2.7, NVIDIA Tesla K40c-class card)
- Article-reported CPU DP reference runtimes (e.g. average 324.16 s at D=5 across the four real-case datasets)

## Outputs

- runtime_benchmark.csv: four dataset rows with columns dataset name, G.A.M.E. runtime (s), CPU DP reference runtime (s), and speed-up factor
- Per-execution wall-clock timings of csccp-solver-cli at dec=5, r=3, averaged over random samplings of cases per dataset

## How to apply

Clone the G.A.M.E. source from github.com/CMDM-Lab/GAME (or the Zenodo deposit doi 10.5281/zenodo.237579) and install its dependencies (Python 2.7, CUDA 7.5, PyCuda 2017.1; the reference hardware is an NVIDIA Tesla K40c with compute capability 3.5, 15 SMs of 192 cores). For each of the four testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.), run csccp-solver-cli over the dataset's seed scaffolds listed in GAME/examples/Datasets/scaffolds with the DP solver flags (-v idp, -l cc), r=3, and --dec 5, passing targeted molecular weights (-m) from GAME/examples/MW/. Measure wall-clock runtime per execution, averaging over random samplings such as 10 samplings of 20 cases per dataset as in the paper's protocol. Retrieve the article-reported CPU DP reference (average 324.16 s at D=5 across the four datasets, taken from the text rather than re-executed) and compute speed-up = CPU DP time / G.A.M.E. time. Write runtime_benchmark.csv with exactly four dataset rows containing dataset name, G.A.M.E. runtime, CPU DP reference runtime, and speed-up factor; expected magnitude is ~20.72 s average for G.A.M.E. at D=5 versus orders-of-magnitude longer CPU times.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver (csccp-solver-cli) whose wall-clock runtime is measured at dec=5 for the four testing datasets) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit underpinning the G.A.M.E. parallelization being benchmarked against the CPU DP baseline)
- **PyCuda 2017.1** (Python binding used by G.A.M.E. to launch the CUDA kernels timed in the benchmark)
- **NVIDIA Tesla K40c** (Reference GPU hardware (compute capability 3.5, 15 SMs × 192 cores) on which benchmark timings are taken)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- runtime_benchmark.csv contains exactly 4 dataset rows (row_count_equals 4), one per testing dataset
- Each G.A.M.E. execution completes within a bounded evaluation window (well under 24 hours), consistent with the reported ~20.72 s average at D=5
- Speed-up factor for each dataset equals CPU DP reference time divided by G.A.M.E. time, with the CPU DP average across the four datasets equal to 324.16 s at D=5
- Solver flags match the protocol: DP solver (-v idp, -l cc), --dec 5, -r 3, and -m values taken from GAME/examples/MW/
- Reported acceleration falls in the paper's range (e.g. an average acceleration factor of 33 in 99% of the cases) rather than orders-of-magnitude outliers

## Limitations

- CPU DP reference times are taken from the article text rather than re-executed, so the speed-up factors inherit any reporting differences in hardware or protocol
- Benchmark timings are hardware-dependent: the reference numbers come from an NVIDIA Tesla K40c (compute capability 3.5); other GPUs will shift absolute G.A.M.E. runtimes
- The DP running time grows by orders of magnitude as decimal digits increase, so results at dec<5 do not extrapolate to higher precision, and worst-case CSCCP can still require exponential time

## Evidence

- [abstract] CUDA-based GPU-accelerated mixture elucidator improves DP performance enabling five-decimal-digit masses: "we designed a "compute unified device architecture" (CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP"
- [methods] Software stack used for the benchmark: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [methods] Reference GPU hardware specification: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [discussion] Reported acceleration factor from the paper: "an average acceleration factor of 33 in 99% of the cases"
- [readme] CLI invocation pattern for the solver: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] Location of the targeted molecular weights used as benchmark inputs: "The targeted molecular weights of the four datasets were also provided in `GAME/examples/MW/`. (the parameter of -m in csccp-solver-cli)"
