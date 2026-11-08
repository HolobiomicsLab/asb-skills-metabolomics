---
name: dp-cost-matrix-truncation
description: Use when running the dynamic-programming phase of G.A.M.E. at high mass
  decimal precision (D=5) on GPU and peak device memory becomes the binding constraint
  rather than compute time.
when_to_use_negative:
- Input masses only need one decimal digit (D=1), where the uncompressed DP already
  runs in acceptable memory on the GPU
- The problem is not a CSCCP instance from a seed scaffold database — there is no
  DP cost matrix to truncate
- The bit-budget constraints cannot be satisfied for a scaffold configuration (Σ⌊log2(k_s)+1⌋
  > 32 or n⌊log2(R)+1⌋ > 32), so packing into two 32-bit integers is not valid
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP implementation whose first phase is memory-optimized by
    discarding per-iteration data and applying the Lc compression technique
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python bindings used to execute the compressed DP kernels on the GPU
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit on which G.A.M.E.'s kernels run
- name: NVIDIA Tesla K40c
  role: Hardware used to benchmark peak global memory of compressed vs. uncompressed
    runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:b04cbbeabe0d38965771269e399b5ad2f7081ed98fa34f7a033dd0f59ff8e633
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
yes_rate: 0.75
n_claims: 12
---

# dp-cost-matrix-truncation

## Summary

A memory-compression technique from G.A.M.E. that reduces GPU global-memory consumption of the DP phase for the chemical substituent core combinatorial problem (CSCCP) by discarding unnecessary cost data after each iteration and packing the sidechain information matrix L into a compressed matrix Lc of two 32-bit integers per (w,r) cell.

## When to use

Use when running the dynamic-programming phase of G.A.M.E. at high mass decimal precision (D=5) on GPU and peak device memory becomes the binding constraint rather than compute time.

## When NOT to use

- Input masses only need one decimal digit (D=1), where the uncompressed DP already runs in acceptable memory on the GPU
- The problem is not a CSCCP instance from a seed scaffold database — there is no DP cost matrix to truncate
- The bit-budget constraints cannot be satisfied for a scaffold configuration (Σ⌊log2(k_s)+1⌋ > 32 or n⌊log2(R)+1⌋ > 32), so packing into two 32-bit integers is not valid

## Inputs

- Seed scaffold file in .cIdx format (e.g. from GAME/examples/core_index)
- Targeted molecular weight (parameter -m, e.g. 168.195105) with decimal precision D (1–5) and results count r (3–10)
- Configuration index (-c) chosen from the configuration count in the .cIdx file
- Scaffold database with sidechain linking probabilities (NPDB-derived, 83,242 scaffolds)

## Outputs

- Compressed matrices C−, C+ (2 × w'max × R) and Lc (two packed 32-bit integers per (w,r)) held in GPU global memory
- Top-r ranked structures with probabilities and SMILES (e.g. top-3 with probabilities like 1.972266e-04)
- Benchmark comparison of peak GPU memory with compression on vs. off

## How to apply

After computing the cost matrix C and sidechain information matrix L in the first DP phase, discard unnecessary data after each iteration and replace full matrices with two sliding sub-matrices C− and C+ of size 2 × w'max × R instead of n × w'max × R (saving 100(1 − 2/n)% memory). Compress L into Lc by packing the n tuples (k_s,w,r, r^prev_s,w,r) per (w,r) into two 32-bit integers under the bit-budget constraints Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32. Convert all mass values to integers by multiplying by 10^D (e.g. D=5) before indexing, implement the compressed run with PyCuda 2017.1 on CUDA 7.5 (tested on an NVIDIA Tesla K40c), and keep the second DP backtracking phase unchanged so the final top-r ranking (e.g. r=3) remains reproducible.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP implementation whose first phase is memory-optimized by discarding per-iteration data and applying the Lc compression technique) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python bindings used to execute the compressed DP kernels on the GPU) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit on which G.A.M.E.'s kernels run)
- **NVIDIA Tesla K40c** (Hardware used to benchmark peak global memory of compressed vs. uncompressed runs)

## Examples

```
$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Peak GPU global memory of the compressed run is strictly lower than the uncompressed baseline on the same scaffold and mass
- Top-3 predicted compounds and their probabilities are identical between compressed and uncompressed runs (e.g. matching the 1.972266e-04 / 1.150488e-04 / 6.574219e-05 reference ranking)
- Bit budgets hold for every (w,r) cell: Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32
- Stored values per (w,r) drop from n × w'max × R to 2 × w'max × R, i.e. the expected 100(1 − 2/n)% saving is observed
- The end-to-end csccp-solver-cli run terminates with the expected 'Finished with 1 iterations' style output and a ranked SMILES list

## Limitations

- Compression is only valid when the per-cell tuple counts fit the 32-bit budgets; scaffolds with many sidechain positions per (w,r) cell may violate them
- Memory savings depend on n (number of sidechain linkages); with small n the 100(1 − 2/n)% saving shrinks
- G.A.M.E. requires GPU hardware with sufficient compute capability (Tesla K40c, compute capability 3.5) and correctly mounted CUDA device paths; the README notes configuring the path and mounting the device in /dev per the CUDA installation guide

## Evidence

- [methods] Memory usage reduction: "The memory usage is reduced by discarding unnecessary data after each iteration and applying a compression technique."
- [methods] Lc compression technique: [evidence span withheld — anchor status: not_found]
- [methods] mass-to-integer conversion: "all mass values must be converted to integers by multiplying 10D in the program"
- [methods] software stack: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] README CLI invocation with r and dec parameters: "PS. In general, r is set to 3~10, dec is set to 1~5"
