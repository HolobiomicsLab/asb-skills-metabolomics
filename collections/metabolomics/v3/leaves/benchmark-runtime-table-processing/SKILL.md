---
name: benchmark-runtime-table-processing
description: Use when a runtime benchmark table containing CPU DP and G.A.M.E. GPU
  DP running times per dataset exists and per-dataset acceleration factors (CPU time
  / GPU time) must be computed and validated against the reported 10–1000x range.
when_to_use_negative:
- Input is raw benchmark timing logs rather than an aggregated per-dataset runtime
  table — aggregation must happen first
- The analysis goal is correctness of G.A.M.E.'s predicted structures (probabilities,
  weights, SMILES) rather than runtime performance comparison
- Benchmark timings were collected at fewer than five decimal digits of mass (D<5),
  where the reported acceleration regime differs
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver whose running times form the denominator of each
    acceleration factor; its CLI is invoked with --dec 5 for five decimal digits
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Scripting environment (with PyCuda) used to load the benchmark table, compute
    CPU/GPU time ratios, and write acceleration_factors.csv
- name: CUDA
  role: GPU programming toolkit (CUDA 7.5) underlying G.A.M.E.'s parallelized DP implementation
    being benchmarked
- name: NVIDIA Tesla K40c
  role: Hardware (compute capability 3.5, 15 SMs of 192 cores each) on which the GPU
    timings are measured
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:61d324e5587b1ad08e955c65cd46b6491683b29d8becccdf476286b2d7192b93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA
  - G.A.M.E.
  - NVIDIA Tesla K40c
  - Python 2.7
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# benchmark-runtime-table-processing

## Summary

Derive per-dataset GPU-over-CPU acceleration factors from the G.A.M.E. runtime benchmark table by dividing the conventional CPU-based DP average running time by the G.A.M.E. GPU DP running time at D=5, R=3, quantifying the speed-up G.A.M.E. achieves over Su's DP algorithm on natural product testing datasets.

## When to use

Use when a runtime benchmark table containing CPU DP and G.A.M.E. GPU DP running times per dataset exists and per-dataset acceleration factors (CPU time / GPU time) must be computed and validated against the reported 10–1000x range.

## When NOT to use

- Input is raw benchmark timing logs rather than an aggregated per-dataset runtime table — aggregation must happen first
- The analysis goal is correctness of G.A.M.E.'s predicted structures (probabilities, weights, SMILES) rather than runtime performance comparison
- Benchmark timings were collected at fewer than five decimal digits of mass (D<5), where the reported acceleration regime differs

## Inputs

- Runtime benchmark CSV with G.A.M.E. GPU DP and CPU DP average running times per dataset at D=5, R=3, wmax from real MS experiments
- Article-reported CPU DP reference average of 324.16 s (vs G.A.M.E. 20.72 s at D=5) for datasets where CPU execution at D=5 was infeasible

## Outputs

- acceleration_factors.csv with one row per testing dataset (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) and its computed acceleration factor

## How to apply

Load the runtime benchmark table (CSV) emitted by the preceding benchmarking card, which contains G.A.M.E. GPU and CPU DP running times per dataset at D=5, R=3, with wmax taken from real MS experiments. For each of the four natural product testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.), compute the acceleration factor as the CPU DP average running time divided by the GPU running time, following the article's definition of the speed-up factor. Where CPU execution at D=5 was infeasible for a dataset, substitute the article-reported CPU DP reference average of 324.16 s against G.A.M.E.'s 20.72 s at D=5 in real cases before computing the ratio. Write the dataset names and factors to acceleration_factors.csv and validate that every factor falls within the one-to-two-orders-of-magnitude (10 to 1000) acceleration range the article reports for G.A.M.E. over Su's CPU DP.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver whose running times form the denominator of each acceleration factor; its CLI is invoked with --dec 5 for five decimal digits) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Scripting environment (with PyCuda) used to load the benchmark table, compute CPU/GPU time ratios, and write acceleration_factors.csv)
- **CUDA** (GPU programming toolkit (CUDA 7.5) underlying G.A.M.E.'s parallelized DP implementation being benchmarked)
- **NVIDIA Tesla K40c** (Hardware (compute capability 3.5, 15 SMs of 192 cores each) on which the GPU timings are measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- All four dataset rows (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) are present in acceleration_factors.csv with non-empty factor fields
- Every computed factor falls within the one-to-two-orders-of-magnitude (10 to 1000) acceleration range reported for G.A.M.E. over Su's CPU DP
- The substituted reference ratio (324.16 s / 20.72 s ≈ 15.6x) for infeasible-CPU datasets also lies within the accepted range
- Dataset names and units in the output CSV match the source benchmark table (average running times, D=5, R=3)
- The average of the four factors is consistent with the article's reported average acceleration factor of 33

## Limitations

- The article reports the acceleration factor only as an average over cases ('an average acceleration factor of 33 in 99% of the cases'), so per-dataset factors may vary substantially from 33
- CPU execution at D=5 was infeasible for at least one dataset, requiring substitution of the article-reported reference average rather than a directly measured ratio
- Timings depend on the specific GPU hardware (NVIDIA Tesla K40c, CUDA 7.5); factors are not transferable to other GPUs without re-benchmarking
- Results reflect the mass-decimal-digit sensitivity of DP running time (tenfold growth per additional decimal digit), so D<5 comparisons will not reproduce these magnitudes

## Evidence

- [discussion] By harnessing the power of modern GPU graphic cards, G.A.M.E. outperforms Su's DP algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] an average acceleration factor of 33 in 99% of the cases: "an average acceleration factor of 33 in 99% of the cases"
- [abstract] considerably improves the performance of the DP, allowing up to five decimal digits: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [methods] Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [readme] csccp-solver-cli command with --dec 5 for five decimal digits: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
