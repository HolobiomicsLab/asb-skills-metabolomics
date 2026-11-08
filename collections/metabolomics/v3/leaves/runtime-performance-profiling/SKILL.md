---
name: runtime-performance-profiling
description: Use when you must profile runtime versus solution quality of alternative
  solvers on a shared benchmark — e.g. timing G-Aligner's Greedy, Gurobi and VLSNS
  solvers across three LC-MS datasets and verifying a reported speedup ratio.
when_to_use_negative:
- When the goal is comparing G-Aligner against external feature alignment software
  (OpenMS, MZmine2, XCMS) rather than profiling its own internal solvers
- When raw mzML files still need feature extraction — profiling assumes G-Aligner
  alignment outputs already exist in the experiment folder
- When only alignment accuracy is of interest and no runtime measurements were recorded,
  since the skill's core output is the runtime-vs-accuracy trade-off table
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Source of the three solver implementations (Greedy, Gurobi, VLSNS with MSG)
    whose runtimes and F_ACC/A_ACC accuracy are profiled; alignment results are saved
    in the experiment folder
  repo: https://github.com/CSi-Studio/G-Aligner
- name: Gurobi
  role: Optimal global solver used as the accuracy and runtime baseline for the comparison
- name: pandas
  role: Loads annotated feature lists and alignment outputs, computes F_ACC/A_ACC,
    aggregates solver metrics and writes solver_comparison.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:4d60980e14b087fae72587a52429adbba063745215e253f64bddbadc25c719cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - Gurobi
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# runtime-performance-profiling

## Summary

Benchmarks the wall-clock running time of G-Aligner's three combinatorial optimization solvers (Greedy, Gurobi, VLSNS/MSG) against their feature alignment accuracy (F_ACC/A_ACC) on the TripleTOF 6600, QE HF and MTBLS562 datasets, producing a per-dataset solver comparison table that quantifies the accuracy-vs-speed trade-off.

## When to use

Use when you must profile runtime versus solution quality of alternative solvers on a shared benchmark — e.g. timing G-Aligner's Greedy, Gurobi and VLSNS solvers across three LC-MS datasets and verifying a reported speedup ratio.

## When NOT to use

- When the goal is comparing G-Aligner against external feature alignment software (OpenMS, MZmine2, XCMS) rather than profiling its own internal solvers
- When raw mzML files still need feature extraction — profiling assumes G-Aligner alignment outputs already exist in the experiment folder
- When only alignment accuracy is of interest and no runtime measurements were recorded, since the skill's core output is the runtime-vs-accuracy trade-off table

## Inputs

- Manually annotated feature lists for the three benchmark datasets (m/z, RT, area information of library analytes)
- G-Aligner alignment outputs for the Greedy, Gurobi and VLSNS (MSG) solvers on the TripleTOF 6600, QE_HF and MTBLS562 datasets
- Recorded wall-clock running time of each solver on each dataset

## Outputs

- solver_comparison.csv with columns: dataset, solver, F_ACC, A_ACC, running time
- Verified relative ordering of the three solvers (accuracy tiers and speedup ratios)

## How to apply

Load the manually annotated feature lists (m/z, RT, area of library analytes) and the G-Aligner alignment outputs for the TripleTOF 6600, QE HF and MTBLS562 datasets with pandas. For each solver — Greedy, the local-search VLSNS solver with MSG, and the Gurobi optimal solver — match aligned consensus features against the manually annotated library analytes to compute F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes). Extract the recorded wall-clock running time of each solver on each dataset from the evaluation outputs, since the multidimensional assignment problem G-Aligner solves is NP-Hard and runtime is its acknowledged main limitation. Aggregate per-dataset metrics into one table with columns dataset, solver, F_ACC, A_ACC and running time, then write solver_comparison.csv. Validate the ordering: Greedy should underperform the global solvers, VLSNS with MSG should reach Gurobi-level accuracy, and VLSNS should be up to 9.6 times faster than Gurobi/Greedy.

## Related tools

- **G-Aligner** (Source of the three solver implementations (Greedy, Gurobi, VLSNS with MSG) whose runtimes and F_ACC/A_ACC accuracy are profiled; alignment results are saved in the experiment folder) — https://github.com/CSi-Studio/G-Aligner
- **Gurobi** (Optimal global solver used as the accuracy and runtime baseline for the comparison)
- **pandas** (Loads annotated feature lists and alignment outputs, computes F_ACC/A_ACC, aggregates solver metrics and writes solver_comparison.csv)

## Examples

```
import pandas as pd; agg = runs.groupby(['dataset','solver'])[['F_ACC','A_ACC','runtime_s']].mean(); agg.to_csv('solver_comparison.csv')
```

## Evaluation signals

- solver_comparison.csv exists and contains one row per dataset × solver combination (3 datasets × 3 solvers) with columns dataset, solver, F_ACC, A_ACC, running time
- Relative ordering reproduced: the Greedy solver's F_ACC/A_ACC values fall below both global solvers on every dataset
- VLSNS (MSG) F_ACC/A_ACC values are at Gurobi-level accuracy on the benchmark datasets
- VLSNS running time is up to 9.6 times faster than the Gurobi and Greedy solvers while keeping alignment within acceptable time
- F_ACC and A_ACC values are reproducible when consensus features are re-matched against the manually annotated library analytes

## Limitations

- The multidimensional assignment problem is NP-Hard, so profiling is sensitive to dataset size and run count; G-Aligner's main limitation is lack of computing speed
- Two acceleration methods keep running time within an acceptable range, but a gap to painless application remains
- GPU acceleration is not yet implemented in G-Aligner
- Optimized solver parameters are reported only in supplementary Tables S1–S3, so reproducing exact runtimes requires fetching those parameter settings

## Evidence

- [discussion] NP-Hard nature of the modeled problem drives runtime profiling: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
- [abstract] Three solvers being profiled: "provides three combinatorial optimization algorithms to find optimal matching solutions"
- [discussion] Runtime kept acceptable but not painless: "Although we provided two acceleration methods to keep the running time within an acceptable range, there was still a gap between painless application."
- [readme] Where evaluation outputs and annotations come from: "Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034"
- [methods] Where G-Aligner alignment outputs are written before profiling: "Feature alignment results are saved in ```experiment``` folder."
