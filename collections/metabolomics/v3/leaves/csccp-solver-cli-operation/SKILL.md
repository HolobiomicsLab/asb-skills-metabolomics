---
name: csccp-solver-cli-operation
description: Use when you need to predict candidate constitutions of an unknown chemical
  mixture from a high-resolution mass value, given seed scaffold files (*.cIdx), a
  scaffold/sidechain probability database, and mass precision up to D=5 decimal digits.
when_to_use_negative:
- The structure of the compound is fully known or MS data alone with low mass precision
  suffices — the CSCCP/DP pipeline is aimed at partially or completely unknown mixture
  constituents at high mass precision
- Only a CPU is available and five decimal digits of mass precision (--dec 5) are
  required — the article reports the CPU DP runtime grows by orders of magnitude as
  decimal digits increase
- The target mass falls outside a scaffold's min_possible_weight/max_possible_weight
  range or no seed scaffold configuration covers the mass window
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: G.A.M.E.
  role: Provides the csccp-solver-cli executable and the GPU-accelerated iterative
    dynamic programming (idp) solver for CSCCP
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python binding used to launch the CUDA idp kernel from the solver
- name: CUDA 7.5
  role: GPU toolkit compiling the parallelized DP kernel executed via PyCuda
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the idp
    kernel is executed
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:d5501cca95a250b0ee5177f8e9a1a4adb0d4abfcad73970e91515c25b54e8bed
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
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# csccp-solver-cli-operation

## Summary

Runs the G.A.M.E. command-line solver (csccp-solver-cli) to solve the chemical substituent core combinatorial problem (CSCCP) via a CUDA/GPU-parallelized iterative dynamic programming (idp) kernel, predicting top-ranked probable molecular structures matching a targeted molecular weight at up to five decimal digits of mass precision.

## When to use

Use when you need to predict candidate constitutions of an unknown chemical mixture from a high-resolution mass value, given seed scaffold files (*.cIdx), a scaffold/sidechain probability database, and mass precision up to D=5 decimal digits.

## When NOT to use

- The structure of the compound is fully known or MS data alone with low mass precision suffices — the CSCCP/DP pipeline is aimed at partially or completely unknown mixture constituents at high mass precision
- Only a CPU is available and five decimal digits of mass precision (--dec 5) are required — the article reports the CPU DP runtime grows by orders of magnitude as decimal digits increase
- The target mass falls outside a scaffold's min_possible_weight/max_possible_weight range or no seed scaffold configuration covers the mass window

## Inputs

- Seed scaffold files in *.cIdx format (e.g. GAME/examples/core_index; dataset-specific scaffold lists in GAME/examples/Datasets/scaffolds)
- Targeted molecular weight value for -m (e.g. from GAME/examples/MW/, e.g. 168.195105)
- Configuration index -c (0 to x-1, where x is the number on line 3 of the *.cIdx file)
- Ranking size -r (top-N structures, typically 3~10)
- Decimal digit precision --dec (1~5)

## Outputs

- Ranked list of predicted structures with probability, weight, and SMILES for each mass peak (e.g. top 3 candidates at the targeted weight 1.681951e+02)
- csccp info block reporting scaffold name, configuration, number of sidechains (K), number of compounds, wmin/wmax, and mass_peak_min/mass_peak_max

## How to apply

Install the G.A.M.E. distribution (git clone https://github.com/CMDM-Lab/GAME.git, compile libgamefft, python setup.py install), then invoke csccp-solver-cli with the seed scaffold folder after -s (*.cIdx files), the target molecular weight after -m, the DP solver (-v idp) in C++ (-v cc), a configuration index -c in [0, x-1] where x is the number on line 3 of the *.cIdx file, top-N ranking (-r, generally 3~10), and decimal digits (--dec, generally 1~5; D=5 requires GPU acceleration). All mass values are converted to integers by multiplying by 10^D inside the program, so five-decimal inputs are feasible only with the CUDA/PyCuda GPU kernel (e.g. on an NVIDIA Tesla K40c); set r to 3~10 and confirm the mass_peak_min/mass_peak_max window derived from ppm tolerates the target. Parse the solver's ranked output (probability, weight, SMILES) per mass peak and aggregate predictions per dataset; validate by checking predicted structures against verified constitutions deposited in GAME/examples/Datasets/structures.

## Related tools

- **G.A.M.E.** (Provides the csccp-solver-cli executable and the GPU-accelerated iterative dynamic programming (idp) solver for CSCCP) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python binding used to launch the CUDA idp kernel from the solver)
- **CUDA 7.5** (GPU toolkit compiling the parallelized DP kernel executed via PyCuda)
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the idp kernel is executed)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- The output block prints a '------------------csccp info----------------' section whose mass_peak (w0) matches the -m input and whose scaffold/configuration fields match the *.cIdx input
- The ranked output lists exactly r structures (e.g. top 3) with Probability, Weight, and Smile columns, and predicted weights equal the targeted molecular weight (e.g. 1.681951e+02)
- The solver reports 'Finished with 1 iterations' when filtered results fall below RR, indicating the DP converged without extra iterations
- For the four natural-product datasets, prediction CSVs match the verified structures in GAME/examples/Datasets/structures with the expected compound counts (5, 7, 7, and 45 constitutions)
- At --dec 5, runtime stays bounded (paper reports ~20.72 s average G.A.M.E. time vs ~324.16 s CPU DP reference), confirming the GPU kernel was actually used

## Limitations

- Five decimal digits of mass precision is impractical on CPU because DP running time grows roughly tenfold per additional decimal digit and by orders of magnitude overall
- Scaffold coverage limits the search: only 26,641 of 83,242 scaffold files from 243,130 NPDB natural products were suitable for testing with wmax = 500
- The mass window (mass_peak_min/mass_peak_max) depends on the ppm tolerance, so borderline masses may exclude the true structure
- PS guidance from the README: r is set to 3~10 and dec to 1~5; deviating outside these ranges is unsupported by the paper's protocol

## Evidence

- [readme] csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5: "The csccp-solver-cli used the DP algorithm (-v idp), implemented in C++ (-v cc)."
- [readme] In the file of *.cIdx, assume that the number in line 3 is x, we can set the index of configuration (-c) from 0 to x-1.: "PS. In general, r is set to 3~10, dec is set to 1~5"
- [methods] all mass values must be converted to integers by multiplying 10D in the program: "all mass values must be converted to integers by multiplying 10D in the program"
- [methods] We used Python 2.7 and CUDA 7.5 in combination with PyCuda 2017.1.: "Our graphic card was an NVIDIA® Tesla K40c with a compute capability of 3.5, equipped with 15 streaming multiprocessors of 192 cores each"
- [abstract] G.A.M.E. allows for efficient and automatic structural elucidation of unknown mixtures for practical: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
