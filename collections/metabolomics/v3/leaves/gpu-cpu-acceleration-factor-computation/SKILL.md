---
name: gpu-cpu-acceleration-factor-computation
description: Use when runtime benchmark tables contain GPU and CPU dynamic programming
  execution times for the same CSCCP tasks (e.g., four natural product datasets at
  D=5) and per-dataset speedup ratios are required.
when_to_use_negative:
- The benchmark only covers D=1 or lower decimal-digit settings where CPU DP runtimes
  are already short and the 10x-1000x D=5 speedup characterization does not apply
- The input contains no paired GPU and CPU timings for the same CSCCP tasks, so a
  ratio cannot be computed without fabricating reference times
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3399
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver whose running times form the numerator-side GPU
    measurements used in the acceleration factor computation
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Scripting environment used with PyCuda 2017.1 to run G.A.M.E. and to compute
    and write the acceleration factors CSV
- name: CUDA 7.5
  role: GPU toolkit driving the parallel DP execution whose timing is compared against
    CPU DP
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores each) on which the
    timed G.A.M.E. runs are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:c45f8452ccd49f66db8c274d3e8d58398dca2c6faac11544057ed101776f02da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA
  - CUDA 7.5
  - G.A.M.E.
  - NVIDIA Tesla K40c
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gpu-cpu-acceleration-factor-computation

## Summary

Computes per-dataset GPU-over-CPU acceleration factors for the G.A.M.E. CUDA-based dynamic programming solver against Su's CPU DP at D=5 decimal digits of mass, quantifying the 10x-1000x speedup reported for CSCCP structure elucidation benchmarks.

## When to use

Use when runtime benchmark tables contain GPU and CPU dynamic programming execution times for the same CSCCP tasks (e.g., four natural product datasets at D=5) and per-dataset speedup ratios are required.

## When NOT to use

- The benchmark only covers D=1 or lower decimal-digit settings where CPU DP runtimes are already short and the 10x-1000x D=5 speedup characterization does not apply
- The input contains no paired GPU and CPU timings for the same CSCCP tasks, so a ratio cannot be computed without fabricating reference times

## Inputs

- Runtime benchmark CSV with G.A.M.E. GPU and CPU DP average running times per dataset at D=5, R=3
- Article-reported CPU DP reference average running time (324.16 s) and G.A.M.E. running time (20.72 s) at D=5 for infeasible CPU cases

## Outputs

- acceleration_factors.csv with four dataset rows (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) and their computed GPU-over-CPU acceleration factors

## How to apply

Load the runtime benchmark CSV emitted by the benchmarking card, which contains G.A.M.E. GPU and CPU DP average running times per dataset at D=5, R=3, wmax from real MS experiments. For each of the four datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.), compute the acceleration factor as CPU DP average running time divided by GPU running time, following the article's speed-up factor definition. Where CPU execution at D=5 was infeasible, substitute the article-reported CPU DP reference average of 324.16 s against G.A.M.E.'s 20.72 s before computing the ratio. Write dataset names and factors to acceleration_factors.csv and validate that each factor falls in the 10-1000 (one-to-two orders of magnitude) range reported for G.A.M.E. over Su's CPU DP.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver whose running times form the numerator-side GPU measurements used in the acceleration factor computation) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Scripting environment used with PyCuda 2017.1 to run G.A.M.E. and to compute and write the acceleration factors CSV)
- **CUDA 7.5** (GPU toolkit driving the parallel DP execution whose timing is compared against CPU DP)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores each) on which the timed G.A.M.E. runs are executed)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- acceleration_factors.csv contains all four expected dataset rows (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) with non-empty factor fields
- Every computed acceleration factor falls within the article-reported 10 to 1000 (one-to-two orders of magnitude) acceleration range
- For datasets where CPU DP at D=5 was infeasible, the substituted ratio 324.16 / 20.72 (~15.6) is used and the substitution is documented
- Each factor is reproducible as CPU average running time divided by GPU average running time from the benchmark CSV rows for the same dataset

## Limitations

- Where CPU execution at D=5 was infeasible, the factor relies on a substituted article-reported reference (324.16 s vs 20.72 s) rather than a measured pair on identical inputs
- Timings are hardware- and software-version specific (NVIDIA Tesla K40c, CUDA 7.5, Python 2.7, PyCuda 2017.1); factors will differ on other GPUs
- The article reports an average acceleration factor of 33 in 99% of the cases, so individual dataset factors can deviate substantially from the mean

## Evidence

- [methods] GPU speedup definition via runtime ratio: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [discussion] Acceleration factor magnitude in 99% of cases: "an average acceleration factor of 33 in 99% of the cases"
- [abstract] GPU acceleration improves DP performance enabling five decimal digits: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits"
- [methods] Software stack for running and timing G.A.M.E.: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [readme] csccp-solver-cli invocation with dec 5 and R 3: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
