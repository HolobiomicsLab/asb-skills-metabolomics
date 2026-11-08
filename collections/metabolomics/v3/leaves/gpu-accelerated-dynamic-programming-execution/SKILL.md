---
name: gpu-accelerated-dynamic-programming-execution
description: Use when you must rank probable chemical structures matching a high-resolution
  target molecular weight from seed scaffolds, especially when mass decimal precision
  (D up to 5) makes a CPU DP solver computationally intractable.
when_to_use_negative:
- Input mass data require more than five decimal digits of precision — the GPU-accelerated
  DP in G.A.M.E. supports only up to --dec 5
- The compounds are partially or completely unknown and no suitable seed scaffold
  exists in the scaffold database, since CSCCP elucidation depends on extending scaffolds
  with sidechains from the curated database
- Only a single pure compound needs structure prediction without a scaffold-based
  sidechain extension formulation, where tools like NP-StructurePredictor directly
  may be more appropriate
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: G.A.M.E.
  role: The GPU-accelerated mixture elucidator itself; its csccp-solver-cli runs the
    iterative dynamic programming kernel (-v idp) on seed scaffolds against target
    masses
  repo: https://github.com/CMDM-Lab/GAME
- name: CUDA 7.5
  role: GPU programming toolkit used to implement and run the parallelized DP kernel
    on the NVIDIA Tesla K40c
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python binding used together with Python 2.7 to launch the CUDA DP kernel
    from the solver
  repo: https://github.com/CMDM-Lab/GAME
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 streaming multiprocessors of 192
    cores each) on which the parallel DP is executed
- name: NP-StructurePredictor
  role: Prior CASE method defining the CSCCP problem formulation that G.A.M.E. solves;
    used as the conceptual basis for scaffold-plus-sidechain prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:0a4813702e4e0c1438b05373ec269f41670c7a2a2db070073e7da5df69b0ab53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA 7.5
  - G.A.M.E.
  - NP-StructurePredictor
  - NVIDIA Tesla K40c
  - PyCuda 2017.1
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gpu-accelerated-dynamic-programming-execution

## Summary

Execute the G.A.M.E. GPU-parallelized iterative dynamic programming solver (csccp-solver-cli with -v idp via CUDA/PyCuda) to solve the NP-complete chemical substituent core combinatorial problem (CSCCP) at up to five decimal digits of mass precision, enabling automatic structure elucidation of unknown mixtures.

## When to use

Use when you must rank probable chemical structures matching a high-resolution target molecular weight from seed scaffolds, especially when mass decimal precision (D up to 5) makes a CPU DP solver computationally intractable.

## When NOT to use

- Input mass data require more than five decimal digits of precision — the GPU-accelerated DP in G.A.M.E. supports only up to --dec 5
- The compounds are partially or completely unknown and no suitable seed scaffold exists in the scaffold database, since CSCCP elucidation depends on extending scaffolds with sidechains from the curated database
- Only a single pure compound needs structure prediction without a scaffold-based sidechain extension formulation, where tools like NP-StructurePredictor directly may be more appropriate

## Inputs

- Seed scaffold core-index files (*.cIdx) from GAME/examples/core_index
- Target molecular weights for the four natural-product datasets from GAME/examples/MW/
- Seed scaffold filename lists per dataset from GAME/examples/Datasets/scaffolds
- Configuration index (-c) selected from 0 to x-1, read from line 3 of the .cIdx file
- Mass precision parameter (--dec, 1-5 decimal digits) and ppm tolerance (e.g. ppm=5)

## Outputs

- Ranked top-k predicted structures per mass peak (probability, weight, SMILES)
- Per-dataset CSV of predictions (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp)
- csccp info block reporting scaffold weight, configuration, sidechain counts (K), and number of compounds

## How to apply

Clone the G.A.M.E. repository, install the libgamefft C++ library and the Python distribution (Python 2.7, CUDA 7.5, PyCuda 2017.1 on an NVIDIA Tesla K40c with compute capability 3.5), then invoke csccp-solver-cli with the seed scaffold files (*.cIdx from GAME/examples/core_index) via -s, the target molecular weight via -m, the scaffold configuration index via -c (valid range 0 to x-1, where x is the count in line 3 of the .cIdx file), the top-k ranking via -r (typically 3-10), and the mass decimal precision via --dec 5, running the iterative dynamic programming kernel (-v idp). Masses are converted to integers by multiplying by 10^D internally; GPU parallelization is over the w-index loop (design a) to maximize parallelism. The solver outputs the top-k ranked structures with probability, weight, and SMILES for each mass peak; aggregate these per dataset (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp) and verify against the verified constitutions in GAME/examples/Datasets/structures. For each mass peak, the CSCCP is generated from the scaffold file and the ppm tolerance (e.g. ppm=5) bounds mass_peak_min and mass_peak_max before the DP runs.

## Related tools

- **G.A.M.E.** (The GPU-accelerated mixture elucidator itself; its csccp-solver-cli runs the iterative dynamic programming kernel (-v idp) on seed scaffolds against target masses) — https://github.com/CMDM-Lab/GAME
- **CUDA 7.5** (GPU programming toolkit used to implement and run the parallelized DP kernel on the NVIDIA Tesla K40c) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python binding used together with Python 2.7 to launch the CUDA DP kernel from the solver) — https://github.com/CMDM-Lab/GAME
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 streaming multiprocessors of 192 cores each) on which the parallel DP is executed)
- **NP-StructurePredictor** (Prior CASE method defining the CSCCP problem formulation that G.A.M.E. solves; used as the conceptual basis for scaffold-plus-sidechain prediction)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index/s0000000001 -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Predicted structure rankings reproduce the verified constitutions in GAME/examples/Datasets/structures for all four datasets (5, 7, 7, and 45 verified compounds for cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp respectively)
- The csccp info block reports expected mass_peak_min/mass_peak_max bounds consistent with the requested ppm (e.g. ±5 ppm around mass_peak) and --dec precision
- The solver completes the iterative DP with a valid convergence reason (e.g. RR=30<=30 with len(filtered_results) < RR) and outputs a top-3 table with monotonic probability ordering
- All reported predicted weights match the target molecular weight to the requested decimal precision (e.g. 1.681951e+02 for -m 168.195105)
- The scaffold_count of suitable scaffolds matches the database expectations (26,641 scaffolds suitable with wmax = 500, 51,562 total configurations)

## Limitations

- Running time of the DP grows about tenfold with each additional decimal digit of mass; five digits is the practical upper bound supported by the GPU acceleration
- Performance depends on the scaffold database coverage: only 26,641 of 83,242 NPDB scaffold files (from 243,130 natural products) were suitable for testing with wmax = 500
- Requires specific GPU hardware/software stack (CUDA 7.5, PyCuda 2017.1, compute capability 3.5 device); benchmarks report an average acceleration factor of 33 in 99% of cases but individual low-parallelism cases gain less

## Evidence

- [abstract] CUDA-based GPU acceleration of the DP allowing five decimal digits: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [abstract] Four verified natural-product testing datasets demonstrate efficient automatic elucidation: "As exemplified by four testing datasets with verified constitutions from natural products, G.A.M.E. allows for efficient and automatic structural elucidation of unknown mixtures for practical"
- [methods] Software stack and hardware versions: "We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1."
- [methods] GPU card specification: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [readme] Example CLI invocation with all key parameters: "csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] Configuration index validity rule from the .cIdx file: "In the file of *.cIdx, assume that the number in line 3 is x, we can set the index of configuration (-c) from 0 to x-1."
- [methods] Scaffold database scale and testable subset: [evidence span withheld — anchor status: not_found]
- [intro] Each additional decimal digit increases DP running time tenfold: "the search for optimal solutions can still be completed in exponential time complexity in the"
