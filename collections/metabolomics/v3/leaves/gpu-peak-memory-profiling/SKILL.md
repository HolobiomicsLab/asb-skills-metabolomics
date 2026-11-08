---
name: gpu-peak-memory-profiling
description: Reference for profiling peak GPU memory of a CUDA/PyCuda DP solver when
  validating that a matrix-compression scheme (e.g., compressed Lc and sliding C−/C+
  sub-matrices) lowers memory usage without altering output predictions.
when_to_use_negative:
- The analysis goal is CPU execution-time speedup or GPU parallelization design (design
  (a) over w values), not memory footprint — the acceleration factor (~33 in 99% of
  cases) is a separate measurement
- Mass precision below the range where DP memory growth matters (e.g., 1 decimal digit),
  where compression overhead buys little
- No NVIDIA GPU with CUDA support is available — the compressed run requires PyCuda/CUDA
  execution on the device
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver whose uncompressed vs. compressed runs are profiled
    for peak memory
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python binding used to launch the CUDA DP kernel and instrument peak global
    memory during execution
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU toolkit with which G.A.M.E. was combined to run the DP on the device
- name: NVIDIA Tesla K40c
  role: Target GPU (compute capability 3.5, 15 streaming multiprocessors of 192 cores
    each) on which peak global memory is recorded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:fbbe030a6e1d6bc35826008c9e78f5d3905d104bdb1f31e8df2058c87f160a75
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
yes_rate: 0.75
n_claims: 8
---

# gpu-peak-memory-profiling

## Summary

Benchmark the peak GPU global-memory footprint of the G.A.M.E. dynamic programming (DP) kernel on an NVIDIA Tesla K40c, contrasting the uncompressed C/L matrix run against the compressed Lc/sliding-submatrix C−/C+ run to verify the designed compression technique actually reduces memory at five decimal digits of mass precision.

## When to use

Reference for profiling peak GPU memory of a CUDA/PyCuda DP solver when validating that a matrix-compression scheme (e.g., compressed Lc and sliding C−/C+ sub-matrices) lowers memory usage without altering output predictions.

## When NOT to use

- The analysis goal is CPU execution-time speedup or GPU parallelization design (design (a) over w values), not memory footprint — the acceleration factor (~33 in 99% of cases) is a separate measurement
- Mass precision below the range where DP memory growth matters (e.g., 1 decimal digit), where compression overhead buys little
- No NVIDIA GPU with CUDA support is available — the compressed run requires PyCuda/CUDA execution on the device

## Inputs

- One seed scaffold .cIdx file from GAME/examples/core_index (e.g., s0000000001) with a configuration index in range [0, x-1]
- Target molecular weight from GAME/examples/MW/ (e.g., 168.195105, mass_peak=414.304985 with ppm=5)
- DP parameters: R=3 (or 3–10), decimal digits D=5 (1–5), top-r=3 results
- Uncompressed baseline reference output (peak memory, top-3 compounds with probabilities)
- GPU environment: Python 2.7, CUDA 7.5, PyCuda 2017.1, NVIDIA Tesla K40c

## Outputs

- Peak GPU global memory (bytes/MB) per run configuration
- memory_benchmark.csv with one row per run (compression on/off) and columns: configuration, peak memory, execution time
- Top-3 predicted compounds with weights, SMILES, and probabilities (to diff against the uncompressed reference)

## How to apply

Load one seed scaffold (.cIdx) with its target molecular weight from the GAME examples and convert all mass values to integers by multiplying by 10^D with D=5 and R=3, since the DP grows ~tenfold per added decimal digit. First execute the uncompressed GPU DP baseline (CUDA 7.5, PyCuda 2017.1, Python 2.7 on the NVIDIA Tesla K40c, compute capability 3.5, 15 SMs × 192 cores), recording peak GPU global memory and the top-3 predicted compounds with probabilities (e.g., via csccp-solver-cli -r 3 --dec 5). Then apply the compression: keep only the two sliding sub-matrices C− and C+ (2 × w'max × R instead of n × w'max × R, saving 100(1 − 2/n)% memory) and pack each L entry — the n (k_s,w,r, r^prev_s,w,r) tuples per (w,r) — into two 32-bit integers under the bit budgets Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32, complementing the discarding of unnecessary data after each iteration. Re-run on the identical scaffold/mass/configuration and record peak global memory again, then assemble a benchmark CSV with one row per run (compression on/off) and columns for configuration, peak memory, and execution time. The profiling is correct only if the compressed peak memory is lower while the top-3 predictions and probabilities remain identical to the uncompressed reference.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver whose uncompressed vs. compressed runs are profiled for peak memory) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python binding used to launch the CUDA DP kernel and instrument peak global memory during execution) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU toolkit with which G.A.M.E. was combined to run the DP on the device)
- **NVIDIA Tesla K40c** (Target GPU (compute capability 3.5, 15 streaming multiprocessors of 192 cores each) on which peak global memory is recorded)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Compressed-run peak GPU global memory is strictly lower than the uncompressed baseline on the same scaffold/mass/configuration
- Compressed-run top-3 predicted compounds and their probabilities are identical to the uncompressed reference output (e.g., the 1.972266e-04 / 1.150488e-04 / 6.574219e-05 ranking for m 168.195105)
- Bit-budget invariants hold: Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 for every packed Lc entry
- memory_benchmark.csv is well-formed with one row per run (compression on/off) and non-empty configuration, peak memory, and execution time columns
- Script runs end-to-end through both phases (cost matrix computation and backtracking) without CUDA errors on the Tesla K40c

## Limitations

- Memory savings depend on scaffold configuration: the 100(1 − 2/n)% saving assumes sliding sub-matrices C−/C+ replace the full n × w'max × R C matrix, so small-n configurations save little
- 32-bit packing constrains scaffold fan-out and result radius: bit budgets Σ⌊log2(k_s)+1⌋ ≤ 32 and n⌊log2(R)+1⌋ ≤ 32 can be violated by scaffolds with many sidechains per position or large n tuples per (w,r)
- Results are hardware-specific (Tesla K40c, compute capability 3.5) and software-version-specific (Python 2.7, CUDA 7.5, PyCuda 2017.1); the README additionally references older CUDA 5 and hadoop-1.0.3 cluster dependencies

## Evidence

- [methods] Memory usage reduction via discarding and compression: "The memory usage is reduced by discarding unnecessary data after each iteration and applying a compression technique."
- [methods] Compressed matrix Lc: [evidence span withheld — anchor status: not_found]
- [methods] Target GPU hardware: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [methods] Software stack: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] CLI run producing top-3 ranking: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
