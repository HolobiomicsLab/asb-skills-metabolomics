---
name: mass-spectrometry-structure-elucidation-benchmarking
description: Use when runtime benchmarking a GPU-accelerated DP structure-elucidation
  pipeline against a CPU DP baseline on natural-product testing datasets (e.g. Cuscuta
  chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) at five decimal
  digits of mass.
when_to_use_negative:
- The input is already a computed acceleration-factor table or published timing comparison
  rather than raw GPU/CPU runtime measurements
- The analysis targets single-compound structure elucidation rather than mixture (CSCCP)
  elucidation from high-resolution mass spectra
- Mass data require more than five decimal digits, beyond G.A.M.E.'s supported range,
  or the DP problem size is small enough that GPU overhead negates speedup
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver for the chemical substituent core combinatorial
    problem (CSCCP) whose running times are benchmarked against the CPU DP baseline
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit enabling the parallelized DP implementation executed
    on the Tesla K40c during benchmarking
- name: Python 2.7
  role: Runtime environment (with PyCuda 2017.1) used to drive G.A.M.E. benchmark
    executions and compute acceleration factors
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which benchmark
    timings were measured
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:c35c8a4d26d14b8e1bd176385d17856bcfdb7640deca06a9acae11ec48542f41
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-spectrometry-structure-elucidation-benchmarking

## Summary

Benchmark the G.A.M.E. GPU-accelerated dynamic programming (DP) algorithm against the conventional CPU DP implementation for computer-assisted mixture structure elucidation from high-resolution mass spectra, quantifying acceleration factors at up to five decimal digits of mass (D=5).

## When to use

Use when runtime benchmarking a GPU-accelerated DP structure-elucidation pipeline against a CPU DP baseline on natural-product testing datasets (e.g. Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) at five decimal digits of mass.

## When NOT to use

- The input is already a computed acceleration-factor table or published timing comparison rather than raw GPU/CPU runtime measurements
- The analysis targets single-compound structure elucidation rather than mixture (CSCCP) elucidation from high-resolution mass spectra
- Mass data require more than five decimal digits, beyond G.A.M.E.'s supported range, or the DP problem size is small enough that GPU overhead negates speedup

## Inputs

- Runtime benchmark CSV with G.A.M.E. GPU DP and CPU DP running times per dataset at D=5, R=3, wmax from real MS experiments
- Seed scaffold files (*.cIdx) from the curated NPDB (83,242 scaffold files from 243,130 natural products; 26,641 suitable for wmax=500 with 51,562 configurations)
- Targeted molecular weights of the four natural-product testing datasets (e.g. from GAME/examples/MW/)

## Outputs

- acceleration_factors.csv with four dataset rows (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) and their GPU-over-CPU speed-up factors
- Per-dataset CPU DP vs G.A.M.E. GPU running-time comparisons at D=5

## How to apply

Load the runtime benchmark table emitted by the benchmarking card, which contains G.A.M.E. GPU and CPU DP running times per dataset at D=5, R=3, and wmax taken from real MS experiments. For each of the four datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.), compute the acceleration factor as CPU DP average running time divided by GPU running time, following the article's speed-up definition. Where CPU execution at D=5 was infeasible, substitute the article-reported CPU DP reference average of 324.16 s (vs G.A.M.E. 20.72 s at D=5 in real cases) before taking the ratio. Run the solver via the csccp-solver-cli with parameters such as -v idp, -r 3 (top 3 structures), and --dec 5 (five decimal digits), on NVIDIA Tesla K40c hardware using Python 2.7, CUDA 7.5 and PyCuda 2017.1. Validate that all four acceleration factors fall within the one-to-two-orders-of-magnitude (10x–1000x) range reported for G.A.M.E. over Su's CPU DP (average acceleration factor of 33 in 99% of cases).

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver for the chemical substituent core combinatorial problem (CSCCP) whose running times are benchmarked against the CPU DP baseline) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit enabling the parallelized DP implementation executed on the Tesla K40c during benchmarking)
- **Python 2.7** (Runtime environment (with PyCuda 2017.1) used to drive G.A.M.E. benchmark executions and compute acceleration factors)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which benchmark timings were measured)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- All four dataset rows and factor fields are present in acceleration_factors.csv
- Each computed acceleration factor falls within the one-to-two-orders-of-magnitude range (10x to 1000x) the article reports for G.A.M.E. over Su's CPU DP
- Substituted CPU reference (324.16 s) paired with G.A.M.E. 20.72 s at D=5 yields a ratio consistent with the ~15.6x factor implied by the reported real-case timings
- Reported average acceleration aligns with the article's stated 'average acceleration factor of 33 in 99% of the cases'
- Solver output for each dataset returns top-3 (R=3) ranked structures whose weights match the targeted molecular weight within the 5 ppm tolerance

## Limitations

- The DP running time grows tenfold with each additional decimal digit of mass, so CPU execution at D=5 can be infeasible and requires a reported reference average (324.16 s) instead of a direct measurement
- The DP algorithm optimizes the NP-complete CSCCP only into pseudo-polynomial time and can still exhibit exponential worst-case behavior
- Speedup depends on GPU hardware: results are specific to an NVIDIA Tesla K40c (compute capability 3.5, 15 streaming multiprocessors of 192 cores) and may not transfer to other cards
- The reported average acceleration factor of 33 applies in 99% of cases, so outlier configurations may fall outside the typical range

## Evidence

- [abstract] CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [discussion] By harnessing the power of modern GPU graphic cards, G.A.M.E. outperforms Su's DP algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] an average acceleration factor of 33 in 99% of the cases: "an average acceleration factor of 33 in 99% of the cases"
- [methods] We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1.: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [methods] verified constitutions from four natural products (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum and Angelica sp.): "verified constitutions from four natural products (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum and Angelica sp.)"
- [readme] csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
