---
name: ab-benchmark-measurement
description: Use when measuring whether a code or algorithm change (e.g., the Lc compressed-matrix
  technique in G.A.M.E.) reduces GPU memory usage on identical inputs, while confirming
  output equivalence against an uncompressed reference run.
when_to_use_negative:
- Input masses need only 1–4 decimal digits and the DP already runs fast — the compression
  complexity is motivated by the tenfold runtime growth per additional mass decimal
  digit at D=5
- No CUDA-capable GPU of compute capability 3.5 is available — the benchmark requires
  the Tesla K40c-class hardware and PyCuda stack
- The goal is accuracy benchmarking against experimentally verified structures rather
  than memory/time A-B comparison of compression on/off
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver whose compressed (Lc, C−/C+) and uncompressed runs
    form the A and B arms of the memory benchmark
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python binding used to execute the CUDA kernels of both benchmark configurations
    and expose GPU global-memory usage
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: CUDA toolkit version used with Python 2.7 to build and run the GPU DP implementations
    being benchmarked
- name: NVIDIA Tesla K40c
  role: Reference GPU hardware (compute capability 3.5, 15 streaming multiprocessors
    of 192 cores) on which peak memory and timing are measured
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:2638ccb9c67aa6ce5512a88e28c1e31d26fbaea95b14c4160e477f6f51f2c6df
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
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ab-benchmark-measurement

## Summary

A/B benchmarking skill for G.A.M.E.'s DP memory-compression: run the same seed scaffold (.cIdx) and targeted mass through both the uncompressed GPU DP baseline and the Lc/C−/C+ compressed variant, recording peak GPU global memory, execution time, and top-3 predictions to verify compression saves memory without altering results.

## When to use

Use when measuring whether a code or algorithm change (e.g., the Lc compressed-matrix technique in G.A.M.E.) reduces GPU memory usage on identical inputs, while confirming output equivalence against an uncompressed reference run.

## When NOT to use

- Input masses need only 1–4 decimal digits and the DP already runs fast — the compression complexity is motivated by the tenfold runtime growth per additional mass decimal digit at D=5
- No CUDA-capable GPU of compute capability 3.5 is available — the benchmark requires the Tesla K40c-class hardware and PyCuda stack
- The goal is accuracy benchmarking against experimentally verified structures rather than memory/time A-B comparison of compression on/off

## Inputs

- Seed scaffold file in .cIdx format from GAME/examples/core_index (e.g. s0000000001)
- Targeted molecular weight from GAME/examples/MW/ (e.g. 168.195105), scaled by 10^D with D=5
- Configuration index (-c) valid from 0 to x-1 where x is the value on line 3 of the .cIdx file
- G.A.M.E. source code from github.com/CMDM-Lab/GAME with PyCuda 2017.1 and CUDA 7.5
- NVIDIA Tesla K40c-class GPU (compute capability 3.5, 15 SMs of 192 cores)

## Outputs

- memory_benchmark.csv with one row per run (compression on/off) and columns for configuration, peak GPU memory, and execution time
- Top-3 predicted compounds with probabilities and SMILES per run (uncompressed reference vs compressed run)
- Measured memory-saving percentage from the C−/C+ sliding sub-matrix reduction (expected ~100(1 − 2/n)% vs n × w'max × R)

## How to apply

Load one seed scaffold (.cIdx file, e.g. s0000000001) with its target molecular weight and convert all mass values to integers by multiplying by 10^D with D=5 decimal digits and R=3 results. Run the uncompressed GPU DP baseline via csccp-solver-cli on the Tesla K40c-class GPU (15 SMs × 192 cores, compute capability 3.5) with Python 2.7 / CUDA 7.5 / PyCuda 2017.1, recording peak global memory, execution time, and the top-3 predicted compounds with probabilities. Then enable the compression module — keeping only the two sliding sub-matrices C− and C+ (2 × w'max × R instead of n × w'max × R, saving 100(1 − 2/n)% memory) and packing each (k_s,w,r, r^prev_s,w,r) tuple into two 32-bit integers under the bit budgets Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 — and rerun on identical inputs. Assemble a benchmark CSV with one row per run (compression on/off) and columns for configuration, peak memory, and execution time, then judge success by lower peak memory in the compressed run and identical top-3 predictions and probabilities.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver whose compressed (Lc, C−/C+) and uncompressed runs form the A and B arms of the memory benchmark) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python binding used to execute the CUDA kernels of both benchmark configurations and expose GPU global-memory usage) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (CUDA toolkit version used with Python 2.7 to build and run the GPU DP implementations being benchmarked)
- **NVIDIA Tesla K40c** (Reference GPU hardware (compute capability 3.5, 15 streaming multiprocessors of 192 cores) on which peak memory and timing are measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Compressed-run peak GPU global memory is strictly lower than the uncompressed baseline peak for the same scaffold/mass/configuration
- Top-3 predicted compounds and their probabilities from the compressed run are identical to the uncompressed reference output
- Benchmark CSV has exactly one row per run (compression on/off) with populated configuration, peak memory, and execution-time columns
- Realized memory saving approximates the theoretical 100(1 − 2/n)% reduction from storing 2 × w'max × R instead of n × w'max × R values
- Bit-budget constraints hold when packing tuples into Lc: Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 for the chosen R=3

## Limitations

- Memory reduction scales as 100(1 − 2/n)% relative to n × w'max × R, so for small n the compression gain shrinks toward zero
- Compression is only justified for the 32-bit packing budgets; scaffolds with many sidechains (large K per configuration, e.g. up to 180 compounds in some configurations) could violate the Σ⌊log2(k_s)+1⌋ ≤ 32 or n⌊log2(R)+1⌋ ≤ 32 constraints
- Only 26,641 of 83,242 scaffold files were suitable for testing with wmax = 500, so benchmark coverage is limited to scaffolds meeting the wmax constraint
- Results are hardware-specific: acceleration and memory figures were measured on a Tesla K40c (compute capability 3.5) and may not transfer to other GPUs

## Evidence

- [methods] Memory usage reduction via discarding and compression: "The memory usage is reduced by discarding unnecessary data after each iteration and applying a compression technique."
- [methods] Lc compression technique description: [evidence span withheld — anchor status: not_found]
- [methods] Software and hardware stack: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [methods] Tesla K40c hardware specification: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [readme] README CLI invocation for a single CSCCP solve: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
