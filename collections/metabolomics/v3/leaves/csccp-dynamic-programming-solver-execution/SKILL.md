---
name: csccp-dynamic-programming-solver-execution
description: Use when a targeted molecular weight (e.g. from GAME/examples/MW) must
  be matched against seed scaffold files (*.cIdx from GAME/examples/Datasets/scaffolds)
  to enumerate and rank top candidate structures via the DP (-v idp) solver.
when_to_use_negative:
- When GPU acceleration is required for high decimal digits — the CPU DP solver's
  running time grows roughly tenfold per additional mass decimal digit, so five-decimal-digit
  runs on large scaffold sets are impractical without G.A.M.E.'s CUDA parallelization
- 'When the scaffold has no suitable configuration: only 26,641 of the 83,242 NPDB
  scaffold files are suitable for testing with wmax = 500, and the targeted mass must
  fall within min/max possible weight for the chosen configuration'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: G.A.M.E.
  role: Distribution providing csccp-solver-cli, which solves the CSCCP via the iterative
    dynamic-programming algorithm (-v idp) to rank structures matching a targeted
    molecular weight
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Runtime environment for installing and running the csccp-solver-cli script
    (python setup.py install)
  repo: https://github.com/CMDM-Lab/GAME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:e051fda8dfc3b4ac0d540b658d5917840959fe72368d36bbaf12f80ec9b04e10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - G.A.M.E.
  - Python 2.7
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# csccp-dynamic-programming-solver-execution

## Summary

Executes the csccp-solver-cli (G.A.M.E. distribution) iterative dynamic-programming solver to identify the most probable molecular structures matching a targeted molecular weight, using seed scaffold files and a chosen configuration index. This operationalizes the CSCCP pseudo-polynomial DP solution from the G.A.M.E. pipeline.

## When to use

Use when a targeted molecular weight (e.g. from GAME/examples/MW) must be matched against seed scaffold files (*.cIdx from GAME/examples/Datasets/scaffolds) to enumerate and rank top candidate structures via the DP (-v idp) solver.

## When NOT to use

- When GPU acceleration is required for high decimal digits — the CPU DP solver's running time grows roughly tenfold per additional mass decimal digit, so five-decimal-digit runs on large scaffold sets are impractical without G.A.M.E.'s CUDA parallelization
- When the scaffold has no suitable configuration: only 26,641 of the 83,242 NPDB scaffold files are suitable for testing with wmax = 500, and the targeted mass must fall within min/max possible weight for the chosen configuration

## Inputs

- Seed scaffold folder with scaffold files (e.g. GAME/examples/normal/example-data, file s0000000001)
- *.cIdx configuration index files (GAME/examples/core_index or GAME/examples/Datasets/scaffolds)
- Targeted molecular weight value (-m), available per dataset in GAME/examples/MW/
- Solver parameters: R (-r, typically 3~10), configuration index (-c), decimal digits (--dec, typically 1~5)

## Outputs

- csccp info block (scaffold name, R, mass_peak w0, mass_peak_min/max, scaffold_weight_rel2config, scaffold_probability_rel2config, wmin/wmax, configuration, number of sidechains K, number of compounds)
- Ranked list of top-R candidate structures with Probability, Weight, and SMILES columns

## How to apply

Load the seed scaffold folder and its *.cIdx index; the number on line 3 of the *.cIdx file bounds the configuration index that can be passed with -c (0 to x-1). Run csccp-solver-cli with -s pointing to the scaffold folder, -m set to the targeted weight (e.g. 168.195105), -v idp for the iterative dynamic-programming C++ solver, -r 3 (R is generally set to 3~10), and --dec set to the mass decimal digits (dec is set to 1~5); all mass values are multiplied by 10^D internally as integers. Read the emitted csccp info block (scaffold weight/probability relative to configuration, wmin/wmax, number of compounds) and the ranked output of Probability, Weight, and SMILES rows. Iteration terminates when RR=30<=30, i.e. when len(filtered_results) < RR. Validate by confirming the CLI runs end-to-end, the reported weights match the targeted mass at the chosen decimal precision, and the returned structures are probable given the scaffold's sidechain linking probabilities.

## Related tools

- **G.A.M.E.** (Distribution providing csccp-solver-cli, which solves the CSCCP via the iterative dynamic-programming algorithm (-v idp) to rank structures matching a targeted molecular weight) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Runtime environment for installing and running the csccp-solver-cli script (python setup.py install)) — https://github.com/CMDM-Lab/GAME

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- The solver completes end-to-end and prints the 'Finished with 1 iterations: RR=30<=30' termination message (or equivalent), indicating len(filtered_results) < RR
- Output Weights in the ranked table match the targeted mass at the chosen --dec precision (e.g. 1.681951e+02 for -m 168.195105 with --dec 5)
- The csccp info block is fully populated: mass_peak_min/max bracket mass_peak, and wmin/wmax are reported (e.g. wmin 61.9890674475, wmax 62.1572625525)
- The configuration index passed with -c is within the valid range 0 to (number on line 3 of the .cIdx file) - 1
- For runtime scaling studies, consecutive-D runtime ratios at D=1-3 fall between 5 and 15, consistent with the reported tenfold-per-digit DP growth

## Limitations

- The CPU DP implementation's runtime grows approximately tenfold with each additional decimal digit of mass, making large D values infeasible without the GPU-accelerated variant
- In general, r is set to 3~10 and dec is set to 1~5; only the GPU-based G.A.M.E. allows up to five decimal digits for input mass data in practice
- Requires libgamefft compiled and installed in /usr/lib (cd lib/libgamefft; make; sudo make install), plus numpy >= 1.7.1, openbabel-python >= 1.3, and Jinja2; GPU support additionally needs cuda 5 and pycuda

## Evidence

- [readme] csccp-solver-cli invocation: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
- [readme] configuration index constraint: "In the file of *.cIdx, assume that the number in line 3 is x, we can set the index of configuration (-c) from 0 to x-1."
- [readme] R and dec conventions: "PS. In general, r is set to 3~10, dec is set to 1~5"
- [methods] integer mass conversion: "all mass values must be converted to integers by multiplying 10D in the program"
- [intro] DP runtime scaling: [evidence span withheld — anchor status: not_found]
- [methods] DP output phase: "the second phase outputs the final ranking of the generated structures in the first phase corresponding to the mass peak"
