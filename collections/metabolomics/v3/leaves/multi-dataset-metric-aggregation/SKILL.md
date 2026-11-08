---
name: multi-dataset-metric-aggregation
description: Use when per-dataset evaluation outputs exist for several benchmark datasets
  and a combined solver-vs-metric table is needed to verify cross-dataset performance
  trends or speedup ratios.
when_to_use_negative:
- Only a single dataset is being evaluated — there is nothing to aggregate across
  datasets and a per-dataset table suffices.
- The inputs are still raw mzML or unaligned feature-extraction outputs; alignment
  must be completed before F_ACC/A_ACC can be computed against the annotated library
  analytes.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: G-Aligner
  role: Source of the per-solver alignment outputs (Greedy, Gurobi, VLSNS) and their
    recorded running times that are aggregated across the three datasets
  repo: https://github.com/CSi-Studio/G-Aligner
- name: pandas
  role: Loads the annotated feature lists and alignment outputs, matches consensus
    features against library analytes, and writes the aggregated solver_comparison.csv
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:9d4d537bbb5fff4d427a13d08e33b5c1a6e7fbd3cb2bc30846a95a9024f17240
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multi-dataset-metric-aggregation

## Summary

Aggregates per-dataset evaluation metrics (F_ACC, A_ACC, solver running time) across the three benchmark datasets (TripleTOF 6600, QE HF, MTBLS562) into a single solver-comparison table, so accuracy–runtime trade-offs of G-Aligner's Greedy, Gurobi and VLSNS solvers can be jointly verified.

## When to use

Use when per-dataset evaluation outputs exist for several benchmark datasets and a combined solver-vs-metric table is needed to verify cross-dataset performance trends or speedup ratios.

## When NOT to use

- Only a single dataset is being evaluated — there is nothing to aggregate across datasets and a per-dataset table suffices.
- The inputs are still raw mzML or unaligned feature-extraction outputs; alignment must be completed before F_ACC/A_ACC can be computed against the annotated library analytes.

## Inputs

- Manually annotated feature lists (m/z, RT, area of library analytes) for TripleTOF 6600, QE HF and MTBLS562 (from Zenodo doi:10.5281/zenodo.8313034)
- G-Aligner alignment outputs per solver (Greedy, VLSNS/MSG, Gurobi) for each of the three datasets, with recorded wall-clock running times
- Evaluation scripts/data layout from the G-Aligner repository data/ folder

## Outputs

- solver_comparison.csv with columns dataset, solver, F_ACC, A_ACC, running_time
- Aggregated speedup ratios (VLSNS vs Gurobi/Greedy) up to 9.6x

## How to apply

Load the manually annotated feature lists and the G-Aligner alignment outputs for the TripleTOF 6600, QE HF and MTBLS562 datasets with pandas, then match aligned consensus features against the manually annotated library analytes (m/z, RT, area) to compute F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes) per solver: Greedy, VLSNS with MSG, and the Gurobi optimal solver. Extract each solver's recorded wall-clock running time per dataset, then melt all per-dataset records into one table with columns dataset, solver, F_ACC, A_ACC and running time. Validate the aggregation by checking the reported relative orderings: Greedy underperforms the global solvers, VLSNS with MSG reaches Gurobi-level accuracy, and VLSNS is up to 9.6 times faster than Gurobi/Greedy. Write the result to solver_comparison.csv for downstream expert review.

## Related tools

- **G-Aligner** (Source of the per-solver alignment outputs (Greedy, Gurobi, VLSNS) and their recorded running times that are aggregated across the three datasets) — https://github.com/CSi-Studio/G-Aligner
- **pandas** (Loads the annotated feature lists and alignment outputs, matches consensus features against library analytes, and writes the aggregated solver_comparison.csv table)

## Examples

```
python data/metapro_result_comparison.py
```

## Evaluation signals

- solver_comparison.csv contains one row per dataset-solver combination: 3 datasets x 3 solvers (Greedy, VLSNS/MSG, Gurobi) = 9 rows.
- Relative ordering check: Greedy's F_ACC and A_ACC fall below both global solvers on every dataset.
- VLSNS-with-MSG accuracy check: VLSNS F_ACC/A_ACC match Gurobi-level values on all three datasets.
- Speedup check: computed VLSNS running-time ratio vs Gurobi/Greedy reproduces the reported up-to-9.6x speedup and all running times are positive finite values.
- Expert review confirms the compiled table reproduces the paper's reported accuracy/runtime trade-off trends.

## Limitations

- G-Aligner's main limitation is lack of computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computing time, so runtime aggregation must retain absolute wall-clock times, not just ratios.
- The two acceleration methods keep runtime within an acceptable range but a gap to painless application remains; aggregated runtimes are dataset-size dependent.
- Optimized evaluation parameters are reported only in supplementary tables (S1-S3), so reproducibility of the aggregated metrics depends on fetching those per-dataset parameter sets.

## Evidence

- [readme] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [readme] Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo]: "Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo]"
- [discussion] G-Aligner achieved the highest feature alignment accuracy on all benchmark datasets with reasonable computational time.: "G-Aligner achieved the highest feature alignment accuracy on all benchmark datasets with reasonable computational time."
- [discussion] G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time.: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
