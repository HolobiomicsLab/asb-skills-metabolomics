---
name: scaffold-sidechain-combinatorial-search
description: Use when you have a target molecular weight (up to five decimal digits)
  from a mass spectrum of an unknown natural-product mixture and a scaffold database
  with sidechain probabilities, and need ranked candidate structures.
when_to_use_negative:
- Mass precision beyond five decimal digits is required, or hardware lacks a CUDA-capable
  GPU for the idp kernel
- The query is a single pure compound rather than an unknown mixture whose constituents
  must be combinatorially assembled from scaffolds plus side chains
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP solver of CSCCP, invoked via its csccp-solver-cli to rank
    candidate structures for a target mass
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU parallelization framework used to run the idp dynamic-programming kernel
    in parallel over w values
- name: PyCuda 2017.1
  role: Python binding used to launch the CUDA DP kernel from Python 2.7
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the parallelized
    DP is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:4e10b4f6dcc9025d18ca4662ed6ce579d9cb404b841b8b2ef8da2517b86fdf47
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# scaffold-sidechain-combinatorial-search

## Summary

Solves the chemical substituent core combinatorial problem (CSCCP) by extending seed scaffold files (*.cIdx) with probabilistic side chains to match a high-resolution target mass, using the GPU-accelerated dynamic programming solver (G.A.M.E., idp kernel) to rank top candidate structures.

## When to use

Use when you have a target molecular weight (up to five decimal digits) from a mass spectrum of an unknown natural-product mixture and a scaffold database with sidechain probabilities, and need ranked candidate structures.

## When NOT to use

- Mass precision beyond five decimal digits is required, or hardware lacks a CUDA-capable GPU for the idp kernel
- The query is a single pure compound rather than an unknown mixture whose constituents must be combinatorially assembled from scaffolds plus side chains

## Inputs

- Seed scaffold files (*.cIdx) from GAME/examples/core_index with configuration count in line 3
- Target molecular weights (GAME/examples/MW/) with up to five decimal digits
- Scaffold configuration index (-c), 0 to x-1 from the .cIdx header
- Mass tolerance parameter (ppm, e.g. ppm=5)

## Outputs

- Ranked list of predicted structures per mass peak with probability, weight, and SMILES (top-r, typically r=3)
- One aggregated prediction CSV per dataset (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp)

## How to apply

Load the seed scaffold filenames (*.cIdx from GAME/examples/core_index, indexed per dataset in GAME/examples/Datasets/scaffolds) and the target molecular weights (GAME/examples/MW/). For each dataset and mass peak, invoke csccp-solver-cli with the scaffold files as input (-s), the target mass (-m), a configuration index -c chosen from 0 to x-1 where x is the number of configurations in line 3 of the .cIdx file, top-k ranking (-r 3 to 10), and --dec 5 to convert mass values to integers by multiplying 10^D; run the GPU-parallelized iterative DP (idp) kernel via CUDA/PyCuda. The solver's DP first phase computes cost matrix C and sidechain information matrix L over four loops indexed by (s,w,r,k), then backtracks in a second phase to output the top-ranked structures with probability, weight, and SMILES. Parse the output and aggregate predictions into one CSV per dataset, then validate against the verified structures deposited in GAME/examples/Datasets/structures.

## Related tools

- **G.A.M.E.** (GPU-accelerated DP solver of CSCCP, invoked via its csccp-solver-cli to rank candidate structures for a target mass) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU parallelization framework used to run the idp dynamic-programming kernel in parallel over w values)
- **PyCuda 2017.1** (Python binding used to launch the CUDA DP kernel from Python 2.7)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the parallelized DP is executed)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Each prediction CSV matches the corresponding verified structures in GAME/examples/Datasets/structures and contains the expected compound identifiers (5, 7, 7, and 45 verified constitutions for the four datasets)
- Solver output reports feasible csccp info: scaffold name, mass_peak, mass_peak_min/max from ppm tolerance, configuration, and number of compounds, before the ranked Probability/Weight/SMILES table
- Predicted weights in the ranked table match the target mass at five decimal places (e.g. 1.681951e+02 for mass_peak 168.195105)
- The DP terminates with the expected iteration condition (e.g. 'Finished with 1 iterations: RR=30<=30' when len(filtered_results) < RR)

## Limitations

- The DP running time grows tenfold with each additional decimal digit of mass; five decimal digits is the practical ceiling enabled by GPU acceleration
- Requires a CUDA-capable GPU; the README notes configuring the CUDA path and mounting the device in /dev
- Only scaffolds present in the database (83,242 scaffolds from 243,130 NPDB natural products; 26,641 suitable with wmax=500, 51,562 configurations) can be recovered, so completely novel cores are missed

## Evidence

- [intro] The chemical substituent core combinatorial problem (CSCCP) defined in the NP-StructurePredictor system has been formally proven to be NP-complete: "The computational problem defined in the NP-StructurePredictor system has been formally proven to be NP-complete; it constitutes a chemical substituent core combinatorial problem (CSCCP)"
- [methods] Mass values converted to integer format with 10^D digits: "all mass values must be converted to integers by multiplying 10D in the program"
- [readme] README example invocation with dec 5 and top-3 ranking: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] Configuration index bounds from .cIdx line 3: "In the file of *.cIdx, assume that the number in line 3 is x, we can set the index of configuration (-c) from 0 to x-1."
- [methods] Scaffold database scale from NPDB: "a total of 83,242 scaffold files were generated from the collected 243,130 natural products in the NPDB"
