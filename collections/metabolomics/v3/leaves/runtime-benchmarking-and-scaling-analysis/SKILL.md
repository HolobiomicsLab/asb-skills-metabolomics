---
name: runtime-benchmarking-and-scaling-analysis
description: Use when you need to empirically characterize how solver runtime scales
  with mass decimal precision D for the DP-based CSCCP solver, or to validate pseudo-polynomial
  (~10x per decimal digit) scaling claims on scaffold/mass inputs.
when_to_use_negative:
- You want to benchmark the GPU-accelerated G.A.M.E. implementation rather than the
  CPU DP solver (-v idp); the tenfold-per-digit scaling is a property of the sequential
  DP.
- The task is structure elucidation itself (ranking candidate molecules by probability/weight)
  rather than runtime characterization; use the solver's normal output mode instead.
- Decimal-digit settings beyond D=3 are required on CPU hardware, since feasible CPU
  DP runs are limited to small D (the GPU version exists precisely because runtime
  grows tenfold per digit).
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_1317
tools:
- name: G.A.M.E.
  role: Provides csccp-solver-cli, whose CPU iterative DP mode (-v idp) is the solver
    being timed at successive -dec (D) settings
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Used to orchestrate solver invocations, time each run, and record per-D runtimes
    into the CSV benchmark table
  repo: https://github.com/CMDM-Lab/GAME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:29ea8324e264289a8b0c7b6167d72e3124d6976c8d30eb284b3ff61b11a8c3b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - G.A.M.E.
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# runtime-benchmarking-and-scaling-analysis

## Summary

Benchmarks the CPU iterative dynamic-programming (DP) solver (csccp-solver-cli -v idp) at successive mass decimal-digit settings (D=1-3) on the Cuscuta chinensis testing dataset to verify the reported ~tenfold-per-digit runtime growth of the pseudo-polynomial DP algorithm for CSCCP.

## When to use

Use when you need to empirically characterize how solver runtime scales with mass decimal precision D for the DP-based CSCCP solver, or to validate pseudo-polynomial (~10x per decimal digit) scaling claims on scaffold/mass inputs.

## When NOT to use

- You want to benchmark the GPU-accelerated G.A.M.E. implementation rather than the CPU DP solver (-v idp); the tenfold-per-digit scaling is a property of the sequential DP.
- The task is structure elucidation itself (ranking candidate molecules by probability/weight) rather than runtime characterization; use the solver's normal output mode instead.
- Decimal-digit settings beyond D=3 are required on CPU hardware, since feasible CPU DP runs are limited to small D (the GPU version exists precisely because runtime grows tenfold per digit).

## Inputs

- Cuscuta chinensis seed scaffold index files (*.cIdx) from GAME/examples/Datasets/scaffolds
- Target molecular weights from GAME/examples/MW (5 verified constitutions, 286.24-478.41 Da)
- Configuration indices from .cIdx files (valid -c range 0 to x-1, x from line 3)

## Outputs

- CSV table with columns: D, runtime_seconds (per-D wall-clock runtimes)
- Fitted per-digit growth factors (ratios of consecutive-D runtimes)
- Pass/fail validation: script_runs (end-to-end execution) and value_in_range (each ratio between 5 and 15)

## How to apply

Load the Cuscuta chinensis seed scaffold files (*.cIdx from GAME/examples/Datasets/scaffolds) and the five verified target molecular weights from GAME/examples/MW (mass range 286.24-478.41 Da). For each target mass, invoke the CPU iterative DP solver via csccp-solver-cli with -v idp, R=3, the configuration index from the .cIdx file (line 3 of the file gives the valid range 0..x-1 for -c), and -dec set successively to D=1, 2, and 3; mass values must be converted to integers by multiplying by 10^D inside the program. Time each invocation with a wall-clock timer and record per-D runtimes (seconds) in a CSV (columns: D, runtime_seconds). Compute the ratio of consecutive-D runtimes as the fitted per-digit growth factor; each ratio falling between 5 and 15 confirms the reported tenfold-per-digit DP scaling.

## Related tools

- **G.A.M.E.** (Provides csccp-solver-cli, whose CPU iterative DP mode (-v idp) is the solver being timed at successive -dec (D) settings) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Used to orchestrate solver invocations, time each run, and record per-D runtimes into the CSV benchmark table) — https://github.com/CMDM-Lab/GAME

## Examples

```
csccp-solver-cli -s GAME/examples/Datasets/scaffolds/<scaffold>.cIdx -m 286.24 -v idp -l cc -r 3 -c 0 --dec 1
```

## Evaluation signals

- The benchmark script runs end-to-end without errors (script_runs).
- Every consecutive-D runtime ratio falls between 5 and 15, consistent with the reported tenfold scaling per added decimal digit (value_in_range).
- The CSV contains one runtime row for each (mass, D) combination at D=1, 2, and 3.
- Runtimes increase monotonically with D for every target mass.
- Solver outputs match the README-documented csccp-solver-cli behavior (csccp info block listing scaffold, R, mass_peak, configuration).

## Limitations

- Feasible settings on CPU are limited to small decimal digits (D=1-3); D beyond that is impractical because DP runtime grows an order of magnitude per digit.
- Per-digit growth ratios of 5-15 are approximate; single-run wall-clock timing can vary with machine load, so ratios near the boundaries may fluctuate.
- The benchmark uses only the five verified Cuscuta chinensis constitutions (286.24-478.41 Da) with R=3, so the scaling estimate may not generalize to other datasets, masses, or R values.

## Evidence

- [intro] each additional decimal digit of mass increasing running time tenfold: [evidence span withheld — anchor status: not_found]
- [methods] all mass values must be converted to integers by multiplying 10D in the program: "all mass values must be converted to integers by multiplying 10D in the program"
- [readme] In general, r is set to 3~10, dec is set to 1~5: "PS. In general, r is set to 3~10, dec is set to 1~5"
- [readme] the targeted molecular weights of the four datasets were also provided in GAME/examples/MW/: "The targeted molecular weights of the four datasets were also provided in `GAME/examples/MW/`. (the parameter of -m in csccp-solver-cli)"
