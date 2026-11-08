---
name: combinatorial-solver-benchmarking
description: Use when a method formulates an optimization problem solvable by multiple
  engines (exact, greedy, local search) and you must quantify which solver gives acceptable
  accuracy within acceptable runtime.
when_to_use_negative:
- Only one solver implementation exists for the optimization problem, so there is
  no accuracy/runtime trade-off to compare
- No ground-truth annotation (e.g. manually annotated library analytes) is available
  to compute accuracy metrics like F_ACC/A_ACC, leaving only runtime comparisons
- The task is feature extraction or peak detection rather than evaluating alignment/matching
  solver quality
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Provides the three combinatorial optimization solvers (Greedy, VLSNS with
    MSG, Gurobi) whose alignment accuracy and runtime are benchmarked on the multipartite-graph
    feature matching problem
  repo: https://github.com/CSi-Studio/G-Aligner
- name: pandas
  role: Loads the manually annotated feature lists and G-Aligner evaluation outputs
    and aggregates per-dataset solver metrics into solver_comparison.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:16bf8a82b0aee245532d8d653f884387128fe03d2e2e5856a34abca0874faf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# combinatorial-solver-benchmarking

## Summary

Benchmarks alternative combinatorial optimization solvers for the same optimization problem (here, G-Aligner's unbalanced multidimensional assignment problem for multi-run LC-MS feature matching) by comparing solution accuracy metrics and wall-clock runtime, quantifying accuracy-vs-speed trade-offs.

## When to use

Use when a method formulates an optimization problem solvable by multiple engines (exact, greedy, local search) and you must quantify which solver gives acceptable accuracy within acceptable runtime.

## When NOT to use

- Only one solver implementation exists for the optimization problem, so there is no accuracy/runtime trade-off to compare
- No ground-truth annotation (e.g. manually annotated library analytes) is available to compute accuracy metrics like F_ACC/A_ACC, leaving only runtime comparisons
- The task is feature extraction or peak detection rather than evaluating alignment/matching solver quality

## Inputs

- Manually annotated feature lists (m/z, RT, area of library analytes) for the three benchmark datasets
- G-Aligner alignment outputs produced per solver (Greedy, VLSNS with MSG, Gurobi) on TripleTOF 6600, QE HF and MTBLS562
- Recorded wall-clock running times per solver per dataset

## Outputs

- solver_comparison.csv with columns dataset, solver, F_ACC, A_ACC, running time
- Speedup ratios of VLSNS versus Gurobi/Greedy and accuracy-level comparison verdicts

## How to apply

Run each solver variant (Greedy, Gurobi exact, VLSNS with MSG local search) on identical inputs — G-Aligner's multipartite-graph feature matching on the TripleTOF 6600, QE HF and MTBLS562 benchmark datasets. Score each solver against ground truth by matching aligned consensus features to the manually annotated feature lists: compute F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes). Extract each solver's recorded wall-clock running time per dataset from the evaluation outputs. Aggregate per-dataset metrics into a single table with columns dataset, solver, F_ACC, A_ACC, running time, then verify reported relative orderings: Greedy underperforms the global solvers, VLSNS with MSG reaches Gurobi-level accuracy, and VLSNS is up to 9.6 times faster than Gurobi and Greedy. The rationale is that the multidimensional assignment problem is NP-Hard, so the exact Gurobi solution is the accuracy ceiling while local search may approach it at far lower cost.

## Related tools

- **G-Aligner** (Provides the three combinatorial optimization solvers (Greedy, VLSNS with MSG, Gurobi) whose alignment accuracy and runtime are benchmarked on the multipartite-graph feature matching problem) — https://github.com/CSi-Studio/G-Aligner
- **pandas** (Loads the manually annotated feature lists and G-Aligner evaluation outputs and aggregates per-dataset solver metrics into solver_comparison.csv)

## Examples

```
python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Relative ordering reproduced: Greedy F_ACC/A_ACC below both global solvers (Gurobi and VLSNS) on each of the three datasets
- VLSNS with MSG achieves Gurobi-level accuracy (comparable F_ACC and A_ACC) across TripleTOF 6600, QE HF and MTBLS562
- VLSNS speedup versus Gurobi and Greedy reaches up to 9.6 times, with running time extracted from evaluation outputs not recomputed inconsistently
- Aggregated table schema validated: one row per (dataset, solver) with non-null F_ACC, A_ACC and running time
- F_ACC and A_ACC computed by matching consensus features against the manually annotated library analytes, checked by expert review

## Limitations

- The multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time, so even accelerated solvers leave a gap to painless application
- G-Aligner's main limitation is lack of computing speed; GPU acceleration is not yet implemented
- Optimized per-dataset parameters are reported only in supplementary tables S1-S3, not the main text

## Evidence

- [results] The VLSNS solver was up to 9.6 times faster than the Gurobi and Greedy solvers while empowering G-Aligner to achieve accurate feature alignment in acceptable time.: [evidence span withheld — anchor status: not_found]
- [discussion] G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time.: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
- [abstract] provides three combinatorial optimization algorithms to find optimal matching solutions: "provides three combinatorial optimization algorithms to find optimal matching solutions"
- [readme] Feature alignment results are saved in ```experiment``` folder.: "Feature alignment results are saved in ```experiment``` folder."
- [methods] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
