---
name: mass-spectrometry-mass-peak-matching
description: Use when you have a target molecular weight (1–5 decimal digits) from
  a high-resolution mass spectrum of an unknown natural-product mixture and need to
  rank candidate structures from seed scaffolds (.cIdx files).
when_to_use_negative:
- The mass has more than five decimal digits — the DP running time grows tenfold per
  additional decimal digit and G.A.M.E. supports only up to D=5
- The input is 3D structural or NMR data rather than a molecular-weight mass peak
  from a high-resolution mass spectrum
- No seed scaffold database or probability information is available for the compound
  class of interest
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: G.A.M.E.
  role: CUDA-based GPU-accelerated DP solver (csccp-solver-cli) that matches mass
    peaks to scaffold/sidechain combinations and ranks candidate structures
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU toolkit used with PyCuda 2017.1 to run the parallelized idp kernel on
    an NVIDIA Tesla K40c
- name: PyCuda 2017.1
  role: Python bindings that launch the GPU-parallelized DP kernel from Python 2.7
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the parallel
    DP is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:2894e56449b80b1750a86b8cb14bb3e29b88d0d53e57e46c754c4c1ef04b7abb
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectrometry-mass-peak-matching

## Summary

Matches high-resolution molecular-weight mass peaks against a scaffold-plus-sidechain database using a GPU-accelerated dynamic programming solver (G.A.M.E. csccp-solver-cli) to rank probable constituent structures of unknown mixtures, supporting up to five decimal digits of mass.

## When to use

Use when you have a target molecular weight (1–5 decimal digits) from a high-resolution mass spectrum of an unknown natural-product mixture and need to rank candidate structures from seed scaffolds (.cIdx files).

## When NOT to use

- The mass has more than five decimal digits — the DP running time grows tenfold per additional decimal digit and G.A.M.E. supports only up to D=5
- The input is 3D structural or NMR data rather than a molecular-weight mass peak from a high-resolution mass spectrum
- No seed scaffold database or probability information is available for the compound class of interest

## Inputs

- Target molecular weight of a mass peak (e.g. 168.195105), with up to 5 decimal digits
- Seed scaffold files (*.cIdx) from GAME/examples/core_index listing configurations and sidechain data
- Dataset-specific scaffold index lists from GAME/examples/Datasets/scaffolds and target weights from GAME/examples/MW/
- Configuration index -c (0 to number_of_configurations-1 for the given scaffold)

## Outputs

- Ranked list of predicted structures per mass peak with probability, weight, and SMILES (top r, typically r=3)
- One aggregated prediction CSV per testing dataset (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp)

## How to apply

Convert all mass values to integers by multiplying by 10^D (up to D=5) before matching, since the DP works in integer mass space. Load seed scaffold files (*.cIdx) from GAME/examples/core_index and target molecular weights from GAME/examples/MW/, then invoke csccp-solver-cli with -s (scaffold folder), -m (target mass), -c (configuration index, valid from 0 to x-1 where x is the number of configurations listed in the .cIdx file), -r 3 for top-3 ranking, -v idp for the iterative DP kernel executed via CUDA/PyCuda, and --dec 5. The solver runs the CSCCP DP in two phases: a cost/information matrix computation parallelized over w values on the NVIDIA Tesla K40c, then a backtracking phase that outputs ranked structures with probability, weight, and SMILES. Parse the per-peak output and aggregate into one CSV per dataset (e.g. cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp) for validation against verified structures.

## Related tools

- **G.A.M.E.** (CUDA-based GPU-accelerated DP solver (csccp-solver-cli) that matches mass peaks to scaffold/sidechain combinations and ranks candidate structures) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU toolkit used with PyCuda 2017.1 to run the parallelized idp kernel on an NVIDIA Tesla K40c)
- **PyCuda 2017.1** (Python bindings that launch the GPU-parallelized DP kernel from Python 2.7)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the parallel DP is executed)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index/s0000000001.cIdx -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Each prediction CSV reproduces the verified structures deposited in GAME/examples/Datasets/structures, with expected compound counts of 5, 7, 7, and 45 verified constitutions for cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, and angelica_sp respectively
- Solver output contains the expected csccp info block: mass_peak (w0) matching -m within the ppm-derived mass_peak_min/mass_peak_max range
- Predicted structure weights match the target mass at the specified decimal precision (e.g. all top-3 weights equal 1.681951e+02 for the 168.195105 example)
- Configuration index passed with -c is within 0..x-1 where x is the number of configurations reported for the scaffold
- Ranked output reports probabilities in descending order and respects the requested top-r count (-r 3)

## Limitations

- Mass precision is limited to five decimal digits; running time grows by an order of magnitude per additional decimal digit in the underlying DP
- CSCCP is NP-complete; the search can still exhibit exponential worst-case complexity despite GPU acceleration (average acceleration factor of ~33 in 99% of cases)
- The method requires a scaffold database with sidechain probability information (e.g. Harn's Natural Product Scaffold database); mass spectrometry alone cannot elucidate partially or completely unknown compound structures
- The README notes dec is generally set to 1~5 and r to 3~10

## Evidence

- [abstract] GPU-accelerated DP supports five decimal digits: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [readme] csccp-solver-cli usage with parameters: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] Configuration index bound from .cIdx file: "In the file of *.cIdx, assume that the number in line 3 is x, we can set the index of configuration (-c) from 0 to x-1."
- [methods] Mass values converted to integer format by 10^D: "all mass values must be converted to integers by multiplying 10D in the program"
- [methods] DP second phase outputs ranking of structures: "the second phase outputs the final ranking of the generated structures in the first phase corresponding to the mass peak"
- [readme] Dataset locations for validation: "The identified(validated) main structures in our four tesing natural products were provided in the `GAME/examples/Datasets/structures`"
