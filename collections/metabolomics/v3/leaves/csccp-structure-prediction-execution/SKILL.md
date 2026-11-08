---
name: csccp-structure-prediction-execution
description: Use when a targeted molecular weight from high-resolution mass spectrometry
  must be matched against a natural-product scaffold database to rank the most probable
  candidate structures of an unknown mixture.
when_to_use_negative:
- The task is GPU memory benchmarking or compression of the L matrix (Lc) — that is
  a separate profiling skill, not a single prediction run
- The mass spectrum contains fragments/MS-MS data rather than a single high-resolution
  targeted molecular weight — G.A.M.E. matches a mass_peak with ppm tolerance, not
  fragmentation patterns
- Required mass precision exceeds five decimal digits — the article reports G.A.M.E.
  only allows up to five decimal digits for input mass data
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: G.A.M.E.
  role: GPU-accelerated DP implementation of the CSCCP solver executed via csccp-solver-cli
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU toolkit underlying the parallelized DP computation
- name: PyCuda 2017.1
  role: Python bindings used with Python 2.7 and CUDA 7.5 to launch GPU kernels
- name: NVIDIA Tesla K40c
  role: Target GPU hardware (compute capability 3.5, 15 streaming multiprocessors
    of 192 cores each)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:3a60ae4562599f55dd1c94e16c4f1308993c7cad11ed83fe6743f2dfc0d07b1b
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# csccp-structure-prediction-execution

## Summary

Execute G.A.M.E.'s GPU-accelerated dynamic programming (DP) solver via the csccp-solver-cli to identify the top-r most probable molecular structures matching a targeted molecular weight from seed scaffold (.cIdx) files. This operationalizes the NP-complete chemical substituent core combinatorial problem (CSCCP) in pseudo-polynomial time on CUDA GPUs at up to 5 decimal digits of mass precision.

## When to use

Use when a targeted molecular weight from high-resolution mass spectrometry must be matched against a natural-product scaffold database to rank the most probable candidate structures of an unknown mixture.

## When NOT to use

- The task is GPU memory benchmarking or compression of the L matrix (Lc) — that is a separate profiling skill, not a single prediction run
- The mass spectrum contains fragments/MS-MS data rather than a single high-resolution targeted molecular weight — G.A.M.E. matches a mass_peak with ppm tolerance, not fragmentation patterns
- Required mass precision exceeds five decimal digits — the article reports G.A.M.E. only allows up to five decimal digits for input mass data

## Inputs

- Seed scaffold file (.cIdx) from GAME/examples/core_index, with number of configurations on line 3 and configuration index -c in range 0 to x-1
- Targeted molecular weight (float, e.g. 168.195105) from GAME/examples/MW/
- Configuration index (-c) and result count (-r, typically 3~10)
- Decimal precision (--dec, 1~5 decimal digits of the mass)

## Outputs

- csccp info block: scaffold name, R, mass_peak (w0), mass_peak_min/max, scaffold weight/probability, wmin/wmax, number of sidechains (K), number of compounds
- Top-r ranked compound list with probability, weight, and SMILES strings (e.g. 1.972266e-04 / 1.681951e+02 / C1C/C(=C\CC)C(=O)CC1CO)

## How to apply

Clone the CMDM-Lab/GAME repository, install libgamefft (C++) and the Python distribution, then invoke csccp-solver-cli with a seed scaffold directory (-s), the targeted molecular weight (-m), a configuration index -c from 0 to (x-1) where x is the value on line 3 of the .cIdx file, a result count -r (typically 3~10), and the decimal precision --dec (typically 1~5; G.A.M.E. supports up to five decimal digits). The DP solver runs in two phases: a first phase computing the cost matrix C and sidechain information matrix L via four imbricated loops indexed by (s,w,r,k), and a second backtracking phase that outputs the final ranking of structures. Mass values are converted to integers by multiplying by 10^D before the DP computation. Output is a probability-ranked table of top compounds with weights and SMILES; iterations terminate early when filtered_results count falls below RR (e.g. 'Finished with 1 iterations: RR=30<=30').

## Related tools

- **G.A.M.E.** (GPU-accelerated DP implementation of the CSCCP solver executed via csccp-solver-cli) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU toolkit underlying the parallelized DP computation)
- **PyCuda 2017.1** (Python bindings used with Python 2.7 and CUDA 7.5 to launch GPU kernels)
- **NVIDIA Tesla K40c** (Target GPU hardware (compute capability 3.5, 15 streaming multiprocessors of 192 cores each))

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- The output header block reports the expected scaffold metadata: R=3, mass_peak matching the -m argument, mass_peak_min/mass_peak_max within the ppm tolerance, and a nonzero number of compounds
- The run terminates with a message like 'Finished with 1 iterations: RR=30<=30. Reason: len(filtered_results)=4 < RR', confirming the backtracking phase converged
- The top-3 compounds all have weights printed as ~1.681951e+02, i.e. matching the targeted weight 168.195105, with descending probabilities
- Verified constitutions for the four testing natural products (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) are available in GAME/examples/Datasets/structures for cross-checking predicted SMILES
- The installation test 'test.py' passes after 'python setup.py install'

## Limitations

- G.A.M.E. supports at most five decimal digits (D=5) for input mass data; DP running time grows by orders of magnitude as decimal digits increase
- MS alone cannot elucidate the structure of partially or completely unknown compounds; prediction requires a scaffold database with sidechain probability information (e.g. Harn's Natural Product Scaffold database)
- Only 26,641 of 83,242 NPDB scaffold files were suitable for testing with wmax = 500; scaffolds whose mass range does not bracket the targeted weight are inapplicable
- README GPU support specifies cuda 5 and pycuda and requires configuring the device path and mounting in /dev per the CUDA installation guide

## Evidence

- [readme] csccp-solver-cli solves CSCCP from scaffold + mass: "The program of csccp-solver-cli (GAME) will use the seed scaffolds included in the sepcified foldder after "-s" (the file in this case is s0000000001) to identified the structures having a targeted"
- [readme] parameter conventions r=3~10, dec=1~5: "PS. In general, r is set to 3~10, dec is set to 1~5"
- [abstract] G.A.M.E. allows up to five decimal digits: "considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [methods] DP phases: cost matrix and backtracking ranking: "the first phase comprises four imbricated loops indexed by (s,w,r,k)"
- [methods] mass values converted to integers: "all mass values must be converted to integers by multiplying 10D in the program"
- [readme] README output example shows ranked compounds: "1.	1.972266e-04	1.681951e+02	C1C/C(=C\CC)C(=O)CC1CO"
