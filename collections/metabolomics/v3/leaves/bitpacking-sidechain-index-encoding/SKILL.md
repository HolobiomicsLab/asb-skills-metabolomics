---
name: bitpacking-sidechain-index-encoding
description: Use when GPU global-memory footprint of the DP cost matrix C and sidechain
  matrix L becomes the bottleneck while solving CSCCP at high mass decimal precision
  (D=5, R=3) on devices like the NVIDIA Tesla K40c.
when_to_use_negative:
- When decimal precision is low (dec = 1-2), the DP matrix fits in GPU memory anyway
  and compression adds complexity without benefit
- When the target runtime platform lacks CUDA support or a Kepler-class GPU, since
  the compression scheme is designed for the PyCuda/CUDA G.A.M.E. implementation
- When solving the CSCCP with the non-GPU C++ DP variant at reduced scope where memory
  is not the limiting resource
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP implementation in which the Lc compression technique and
    C-/C+ sliding sub-matrices are embedded
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python binding used to execute the compressed DP kernels on the GPU
- name: CUDA 7.5
  role: CUDA toolkit used with Python 2.7 to build the G.A.M.E. GPU parallelization
- name: NVIDIA Tesla K40c
  role: Target GPU (compute capability 3.5, 15 x 192 cores) on which peak global-memory
    reduction from Lc packing is measured
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:41d444af61279ca6bac817d8f35df80cd8562faacfc559938e8ba9118d9ca35e
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# Bitpacking sidechain index encoding

## Summary

A memory-compression technique from G.A.M.E. that packs the dynamic-programming sidechain information matrix L into a compressed matrix Lc, storing each (k_s,w,r, r^prev_s,w,r) tuple as bit-packed 32-bit integers, so GPU DP execution remains feasible at five decimal digits of mass precision.

## When to use

Use when GPU global-memory footprint of the DP cost matrix C and sidechain matrix L becomes the bottleneck while solving CSCCP at high mass decimal precision (D=5, R=3) on devices like the NVIDIA Tesla K40c.

## When NOT to use

- When decimal precision is low (dec = 1-2), the DP matrix fits in GPU memory anyway and compression adds complexity without benefit
- When the target runtime platform lacks CUDA support or a Kepler-class GPU, since the compression scheme is designed for the PyCuda/CUDA G.A.M.E. implementation
- When solving the CSCCP with the non-GPU C++ DP variant at reduced scope where memory is not the limiting resource

## Inputs

- Seed scaffold file in .cIdx format from GAME/examples/core_index (with configuration index parameter -c from 0 to x-1)
- Targeted molecular weight (e.g. mass_peak=414.304985, or 168.195105 in the README example) with ppm tolerance (e.g. ppm=5)
- Decimal precision parameter dec (1-5, typically 5) and result count r (typically 3-10)
- Harn's Natural Product Scaffold database with sidechain linking probabilities derived from the 83,242 NPDB scaffolds

## Outputs

- Compressed sidechain information matrix Lc holding bit-packed (k_s,w,r, r^prev_s,w,r) tuples as two 32-bit integers per (w,r) cell
- Sliding sub-matrices C- and C+ of size 2 x w'max x R replacing the full n x w'max x R cost matrix
- Top-3 ranked structures with probabilities and SMILES, identical to the uncompressed DP output
- Peak GPU global-memory measurements per run (compression on/off) for a memory_benchmark.csv

## How to apply

First convert all mass values to integers by multiplying by 10^D (D=5) and run the uncompressed GPU DP (CUDA 7.5 + PyCuda 2017.1 under Python 2.7) to record baseline peak GPU memory and the top-3 predicted structures with probabilities. Then, during the first DP phase's four imbricated (s,w,r,k) loops, keep only the two sliding sub-matrices C- and C+ instead of the full n x w'max x R arrays, which saves 100(1 - 2/n)% of memory, and pack the n (k_s,w,r, r^prev_s,w,r) tuples per (w,r) cell into two 32-bit integers in Lc under the bit budgets SUM(floor(log2(k_s))+1) <= 32 and n*floor(log2(R)+1) <= 32. Discard unnecessary data from the DP matrices after each iteration. Run the compressed version on the same seed scaffold (.cIdx file) and targeted molecular weight, e.g. via csccp-solver-cli from the CMDM-Lab/GAME distribution on a Tesla K40c (compute capability 3.5, 15 streaming multiprocessors of 192 cores). Validate that peak global memory decreases while the top-3 output ranking and probabilities remain identical to the uncompressed reference.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP implementation in which the Lc compression technique and C-/C+ sliding sub-matrices are embedded) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python binding used to execute the compressed DP kernels on the GPU)
- **CUDA 7.5** (CUDA toolkit used with Python 2.7 to build the G.A.M.E. GPU parallelization)
- **NVIDIA Tesla K40c** (Target GPU (compute capability 3.5, 15 x 192 cores) on which peak global-memory reduction from Lc packing is measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Compressed-run peak GPU global memory must be strictly lower than the uncompressed DP baseline on the same scaffold/mass input
- Top-3 predicted compounds and their probabilities from the compressed run must be identical to the uncompressed reference output
- Bit-budget constraints must hold: SUM(floor(log2(k_s))+1) <= 32 and n*floor(log2(R)+1) <= 32 for every packed (w,r) cell
- Memory saving should approach 100(1 - 2/n)% for the cost-matrix sub-matrices when n is large
- The csccp-solver-cli run must complete end-to-end and reproduce expected ranking output (e.g. the README example's top structure at probability 1.972266e-04 for mass 168.195105)

## Limitations

- Compression correctness depends on the bit budgets being satisfiable; scaffolds/configurations with many sidechains (large K, e.g. configurations with up to 180 compounds) may stress the 32-bit packing limits
- Validation was specific to the Tesla K40c and CUDA 7.5/PyCuda 2017.1/Python 2.7 stack; the README lists older dependencies (cuda 5, hadoop-1.0.3) that may complicate porting
- Sliding sub-matrix strategy saves memory only asymptotically (100(1 - 2/n)%); for small n the benefit is limited

## Evidence

- [methods] Lc compression technique: [evidence span withheld — anchor status: not_found]
- [methods] Memory usage reduction: "The memory usage is reduced by discarding unnecessary data after each iteration and applying a compression technique."
- [methods] Software stack: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [methods] GPU hardware: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [readme] CLI invocation: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
