---
name: gpu-memory-usage-compression
description: Use when running the GPU DP (G.A.M.E.) at D=5 decimal digits of mass
  precision and peak GPU memory from the full n × w'max × R matrices becomes the bottleneck
  instead of compute time.
when_to_use_negative:
- Running at low mass decimal precision (D=1–2) where the uncompressed DP already
  fits in GPU memory and compression adds complexity without benefit
- When backtracking requires retaining the full L history rather than only the current/previous
  iteration — the sliding C−/C+ scheme discards data after each iteration
- When the sidechain count k_s or R=3 tuple packing violates the 32-bit bit-budget
  constraints (Σ⌊log2(k_s)+1⌋ ≤ 32 or n⌊log2(R)+1⌋ ≤ 32)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: G.A.M.E.
  role: CUDA-based GPU-accelerated DP implementation whose C and L matrices are compressed
    via Lc and the C−/C+ sliding scheme
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU toolkit used to build and execute the memory-compressed DP kernels
- name: PyCuda 2017.1
  role: Python binding used to launch the compressed DP kernels on the GPU alongside
    Python 2.7
- name: NVIDIA Tesla K40c
  role: GPU (compute capability 3.5, 15 SMs × 192 cores) on which peak global memory
    of compressed vs. uncompressed runs is measured
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:7fb54fc4926b4cce00430554992f34e59f072630918bb26a04cd57adee725ebc
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gpu-memory-usage-compression

## Summary

A GPU memory-compression technique from G.A.M.E. that reduces dynamic programming (DP) memory consumption at high mass decimal precision by keeping only two sliding sub-matrices C− and C+ and packing sidechain tuples into a compressed matrix Lc of 32-bit integers.

## When to use

Use when running the GPU DP (G.A.M.E.) at D=5 decimal digits of mass precision and peak GPU memory from the full n × w'max × R matrices becomes the bottleneck instead of compute time.

## When NOT to use

- Running at low mass decimal precision (D=1–2) where the uncompressed DP already fits in GPU memory and compression adds complexity without benefit
- When backtracking requires retaining the full L history rather than only the current/previous iteration — the sliding C−/C+ scheme discards data after each iteration
- When the sidechain count k_s or R=3 tuple packing violates the 32-bit bit-budget constraints (Σ⌊log2(k_s)+1⌋ ≤ 32 or n⌊log2(R)+1⌋ ≤ 32)

## Inputs

- Seed scaffold file (.cIdx) with configuration index and its target molecular weight (e.g. from GAME/examples/core_index and GAME/examples/MW/)
- Mass decimal precision parameter D (e.g. 5) and number of top results r (e.g. 3)
- Sidechain probability database derived from the NPDB (83,242 scaffolds from 243,130 natural products)
- Uncompressed GPU DP baseline run (peak memory and top-3 predictions) for comparison

## Outputs

- Compressed matrix Lc of 32-bit integers holding packed (k_s,w,r, r^prev_s,w,r) tuples
- Two sliding sub-matrices C− and C+ of size 2 × w'max × R
- Top-3 ranked compounds with probabilities and SMILES, identical to the uncompressed baseline
- Memory benchmark (e.g. memory_benchmark.csv) with per-run configuration, peak GPU memory, and execution time

## How to apply

Convert all mass values to integers by multiplying by 10^D (e.g. D=5) and run the DP over (s,w,r,k) loops on the Tesla K40c-class GPU via PyCuda 2017.1 / CUDA 7.5. Instead of storing the full cost matrix C and sidechain information matrix L of size n × w'max × R, discard unnecessary data after each iteration and retain only the two sliding sub-matrices C− and C+ (2 × w'max × R values), saving 100(1 − 2/n)% memory. Pack the n (k_s,w,r, r^prev_s,w,r) tuples per (w,r) cell into two 32-bit integers in the compressed matrix Lc, respecting bit budgets Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 (e.g. R=3). Then re-run the backtracking phase to confirm the compressed run reproduces the uncompressed top-3 ranked structures and probabilities, and log peak global memory for both configurations into a benchmark table.

## Related tools

- **G.A.M.E.** (CUDA-based GPU-accelerated DP implementation whose C and L matrices are compressed via Lc and the C−/C+ sliding scheme) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU toolkit used to build and execute the memory-compressed DP kernels)
- **PyCuda 2017.1** (Python binding used to launch the compressed DP kernels on the GPU alongside Python 2.7)
- **NVIDIA Tesla K40c** (GPU (compute capability 3.5, 15 SMs × 192 cores) on which peak global memory of compressed vs. uncompressed runs is measured)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index/s0000000001.cIdx -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Compressed-run peak GPU global memory is strictly lower than the uncompressed baseline; expected saving approaches 100(1 − 2/n)% for the C/L matrices
- Top-3 predicted compounds, their probabilities, and SMILES are identical between compressed and uncompressed runs on the same .cIdx scaffold and target mass
- Bit budgets hold for every packed (w,r) cell: Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 (R=3)
- Lc stores exactly two 32-bit integers per (w,r) cell, not n raw tuples, confirming correct tuple packing
- Benchmark table contains one row per run (compression on/off) with matching scaffold, mass, configuration, and recorded execution time

## Limitations

- Memory is reduced by discarding unnecessary data after each iteration, so only the current and previous iteration state (C−/C+) is available; access to older DP state is not possible
- Compression is bounded by the 32-bit packing constraints, which limit how many sidechain entries n or scaffold counts k_s can be packed per cell
- The DP running time still grows roughly tenfold per additional decimal digit of mass; compression reduces memory but does not remove this scaling, and acceleration is hardware-dependent (e.g. average factor of 33 on the tested GPU)

## Evidence

- [methods] Memory usage reduction: "The memory usage is reduced by discarding unnecessary data after each iteration and applying a compression technique."
- [methods] Lc compression technique: [evidence span withheld — anchor status: not_found]
- [methods] mass conversion to integers: "all mass values must be converted to integers by multiplying 10D in the program"
- [methods] software stack: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] README CLI example: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
