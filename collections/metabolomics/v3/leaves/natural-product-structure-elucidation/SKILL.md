---
name: natural-product-structure-elucidation
description: Use when predicting mixture constituents from high-resolution MS molecular
  weights of natural products (e.g. Cuscuta chinensis, Ophiopogon japonicus, Polygonum
  multiflorum, Angelica sp.) using scaffold databases with sidechain probabilities.
when_to_use_negative:
- When the task is only benchmarking DP runtime scaling (use the CPU DP at D=1-3 instead
  of the GPU elucidation workflow)
- When no seed scaffold database with sidechain linking probabilities is available
  — the CSCCP formulation depends on scaffold files with configurations and sidechain
  probabilities
- When masses require more than five decimal digits of precision, beyond what G.A.M.E.
  supports
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver (csccp-solver-cli) invoked with -v idp to rank candidate
    structures for each mass peak
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit enabling parallelization of the DP first phase over
    w values
- name: PyCuda 2017.1
  role: Python binding used with CUDA to run the idp kernel on the Tesla K40c
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) executing the parallelized
    CSCCP tasks
- name: Python 2.7
  role: Runtime environment for the csccp-solver-cli and PyCuda execution
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:107efbe6bcb75db95bae0f42454bed812392b6f2fa4dd1863c02c9e93c00bcf0
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# natural-product-structure-elucidation

## Summary

GPU-accelerated dynamic-programming (G.A.M.E.) elucidation of natural-product mixture constituents from high-resolution mass spectral data by extending seed scaffolds with probable sidechains (CSCCP), supporting up to five decimal digits of mass.

## When to use

Use when predicting mixture constituents from high-resolution MS molecular weights of natural products (e.g. Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) using scaffold databases with sidechain probabilities.

## When NOT to use

- When the task is only benchmarking DP runtime scaling (use the CPU DP at D=1-3 instead of the GPU elucidation workflow)
- When no seed scaffold database with sidechain linking probabilities is available — the CSCCP formulation depends on scaffold files with configurations and sidechain probabilities
- When masses require more than five decimal digits of precision, beyond what G.A.M.E. supports

## Inputs

- Target molecular weights from GAME/examples/MW/ (e.g. mass_peak=414.304985 with ppm=5 tolerance)
- Seed scaffold files (*.cIdx) from GAME/examples/core_index, listed per dataset in GAME/examples/Datasets/scaffolds
- Configuration index (-c) read from line 3 of each .cIdx file
- Mass decimal-digit setting (--dec 1-5)

## Outputs

- Ranked predicted structures per mass peak (probability, weight, SMILES), top-3 by default
- Per-dataset prediction CSVs (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp) matching GAME/examples/Datasets/structures

## How to apply

Clone CMDM-Lab/GAME, build libgamefft, and install the csccp-solver-cli. For each target molecular weight in GAME/examples/MW/, select the seed scaffold *.cIdx file listed in GAME/examples/Datasets/scaffolds and its configuration index (line 3 of the .cIdx gives the configuration count; pass -c in 0..x-1). Invoke the iterative dynamic programming kernel (-v idp) — the GPU-parallelized version on an NVIDIA Tesla K40c via CUDA 7.5/PyCuda 2017.1 — with top-3 to top-10 ranking (-r 3) and mass precision --dec 5 (values 1-5), after multiplying masses by 10^D for integer conversion. The DP first phase computes cost matrix C and sidechain information matrix L over loops indexed (s,w,r,k); the second phase backtracks to output the ranked structures (probability, weight, SMILES). Aggregate per-dataset CSV predictions and compare against verified structures in GAME/examples/Datasets/structures.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver (csccp-solver-cli) invoked with -v idp to rank candidate structures for each mass peak) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit enabling parallelization of the DP first phase over w values)
- **PyCuda 2017.1** (Python binding used with CUDA to run the idp kernel on the Tesla K40c)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) executing the parallelized CSCCP tasks)
- **Python 2.7** (Runtime environment for the csccp-solver-cli and PyCuda execution)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index/s0000000001 -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Each dataset's prediction CSV contains the expected number of verified constitutions (5, 7, 7, and 45 for Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.)
- Predicted SMILES match deposited verified structures in GAME/examples/Datasets/structures
- Solver output shows the csccp info block (scaffold, R=3, mass_peak w0, configuration, number of sidechains K) consistent with the .cIdx configuration index used
- Consecutive-D runtime ratios of the DP fall between 5 and 15, confirming the tenfold-per-decimal-digit scaling at D=1-3
- Ranked output lists top-r compounds with monotonically decreasing probabilities at the targeted molecular weight

## Limitations

- CSCCP is NP-complete; DP running time grows tenfold per additional mass decimal digit, so --dec beyond 5 is impractical
- Only 26,641 of 83,242 scaffold files are suitable for testing with wmax = 500; scaffolds outside the mass range are unusable
- Memory must be managed via the compressed matrix Lc and discarding unnecessary data after each iteration; requires CUDA-enabled hardware with device mounted in /dev

## Evidence

- [abstract] G.A.M.E. enables efficient and automatic structural elucidation: "As exemplified by four testing datasets with verified constitutions from natural products, G.A.M.E. allows for efficient and automatic structural elucidation of unknown mixtures for practical"
- [methods] GPU parallelization design: "we implemented the GPU parallelization based on the design (a) because it enabled the highest degree of parallelism"
- [methods] Mass integer conversion: "all mass values must be converted to integers by multiplying 10D in the program"
- [readme] csccp-solver-cli example invocation: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] Seed scaffold and MW file locations: "All seed scaffolds (*.cIdx files) in our collected database that can be used as input of the "csccp-solver-cli" program were provided in the `GAME/examples/core_index`"
