---
name: gpu-kernel-execution-timing
description: Use when comparing GPU vs CPU runtime of a solver whose cost grows steeply
  with input precision (e.g. DP search time increasing tenfold per added mass decimal
  digit), to quantify the speed-up factor at a fixed setting such as dec=5.
when_to_use_negative:
- The goal is structure elucidation accuracy rather than timing — use the solver's
  output ranking directly instead of this benchmarking skill
- The algorithm has no CPU reference time to compare against, so a speed-up factor
  cannot be computed
- Input mass precision is below five decimal digits, where CPU DP runtimes are already
  practical and the GPU comparison is uninformative
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver whose executions are timed via csccp-solver-cli
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: CUDA toolkit used to run the GPU-parallelized DP implementation
- name: PyCuda 2017.1
  role: Python binding executing the timed GPU kernels
- name: NVIDIA Tesla K40c
  role: Reference GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which
    runtimes are measured
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:705f60ca7ca83ea1eaf00e118e163dd61f286d1922fc361f6c017c78c8c2333b
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gpu-kernel-execution-timing

## Summary

Benchmarks wall-clock execution time of a GPU-accelerated algorithm against a CPU reference, as done for G.A.M.E. (CUDA-based dynamic programming mixture elucidator) vs. the CPU DP algorithm at five decimal digits of mass precision across four natural-product testing datasets.

## When to use

Use when comparing GPU vs CPU runtime of a solver whose cost grows steeply with input precision (e.g. DP search time increasing tenfold per added mass decimal digit), to quantify the speed-up factor at a fixed setting such as dec=5.

## When NOT to use

- The goal is structure elucidation accuracy rather than timing — use the solver's output ranking directly instead of this benchmarking skill
- The algorithm has no CPU reference time to compare against, so a speed-up factor cannot be computed
- Input mass precision is below five decimal digits, where CPU DP runtimes are already practical and the GPU comparison is uninformative

## Inputs

- G.A.M.E. source and datasets (github.com/CMDM-Lab/GAME, Zenodo doi 10.5281/zenodo.237579)
- Seed scaffold index files (*.cIdx) under GAME/examples/core_index
- Targeted molecular weights for the four datasets under GAME/examples/MW/
- Article-reported CPU DP reference runtimes (e.g. average 324.16 s at D=5)

## Outputs

- runtime_benchmark.csv with exactly four dataset rows: dataset name, G.A.M.E. (GPU) runtime, CPU DP reference runtime, and speed-up factor

## How to apply

Clone the reference implementation (github.com/CMDM-Lab/GAME) plus its datasets from the Zenodo deposit (doi 10.5281/zenodo.237579), which package seed scaffold files (*.cIdx) and targeted molecular weights under GAME/examples/MW/ and GAME/examples/Datasets/scaffolds. Run the csccp-solver-cli over each of the four testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) with the DP solver (-v idp, -l cc), dec=5 and r=3, passing each dataset's mass values via -m. Measure wall-clock runtime of each execution, averaging over random samplings of cases per dataset as in the paper's protocol (e.g. 10 samplings of 20 cases per dataset) on the reference GPU (NVIDIA Tesla K40c with compute capability 3.5, 15 SMs of 192 cores each). Retrieve the CPU DP reference times from the article text (average 324.16 s at D=5) rather than re-executing, and compute speed-up as CPU DP time divided by GPU time. Write a single CSV (runtime_benchmark.csv) with exactly four dataset rows containing dataset name, GPU runtime, CPU reference runtime, and speed-up factor.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver whose executions are timed via csccp-solver-cli) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (CUDA toolkit used to run the GPU-parallelized DP implementation)
- **PyCuda 2017.1** (Python binding executing the timed GPU kernels)
- **NVIDIA Tesla K40c** (Reference GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which runtimes are measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- runtime_benchmark.csv has row_count_equals 4 (one row per testing dataset)
- Each G.A.M.E. execution completes well within a bounded evaluation window (reported ~20.72 s average at D=5 vs CPU DP ~324.16 s average)
- Speed-up factor per dataset equals CPU DP reference time divided by G.A.M.E. runtime, averaged over samplings (e.g. 10 samplings of 20 cases per dataset)
- All runs use identical solver parameters (dec=5, r=3, -v idp, -l cc) and the dataset-specific -m mass values from GAME/examples/MW/

## Limitations

- GPU runtimes are hardware-dependent: results were obtained on an NVIDIA Tesla K40c and will not transfer directly to other GPUs
- CPU reference times are taken from the article text rather than re-executed, so hardware differences can bias the speed-up factor
- The README documents older dependencies (CUDA 5, pycuda) than the paper's CUDA 7.5/PyCuda 2017.1, so environment setup may need reconciliation

## Evidence

- [abstract] CUDA-based GPU-accelerated mixture elucidator enables five-decimal-digit DP timing benchmark: "we designed a “compute unified device architecture” (CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP"
- [intro] Motivation: DP runtime grows tenfold per added decimal digit of mass: "the search for optimal solutions can still be completed in exponential time complexity in the"
- [methods] Benchmark hardware and software stack for timed executions: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] Concrete CLI invocation used to time the solver: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
