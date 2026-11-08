---
name: lc-ms-feature-alignment-evaluation
description: Use when you need to quantitatively evaluate how accurately a feature
  alignment method (e.g. G-Aligner, MZmine2, OpenMS, XCMS) aligns consensus features
  across multiple LC-MS runs against a manually annotated reference.
when_to_use_negative:
- You only have raw LC-MS data (e.g. mzML files) and still need feature extraction
  — evaluation requires already-extracted feature lists with m/z, RT and area columns
- You want to evaluate feature detection or quantification accuracy rather than alignment
  accuracy across runs
- Your dataset lacks manually annotated library analytes, so TP/FP/F_ACC/A_ACC cannot
  be computed
edam_operation: http://edamontology.org/operation_3807
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: G-Aligner
  role: Produces the feature alignment results (consensus features) whose accuracy
    is evaluated; also benchmarked as the method under test with solvers Gurobi, Greedy,
    VLSNS_MSR and VLSNS_MSG
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison baseline whose built-in feature alignment accuracy is evaluated
    against manual annotation
- name: OpenMS
  role: Comparison baseline evaluated via evaluate_metapro_openms.py
- name: XCMS
  role: Comparison baseline evaluated via evaluate_metapro_xcms.Rmd
- name: Python
  role: Runs evaluation scripts (evaluate_metapro_galigner.py, evaluate_mzmine2_galigner.py,
    evaluate_openms_galigner.py, metapro_result_comparison.py) and aggregates metrics
    with pandas/numpy
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:8434ba807e0aa875f10e4bbe03e7572eb8ecdba46c3dbf141c3f485408ab59a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - Python
  - XCMS
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3807
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lc-ms-feature-alignment-evaluation

## Summary

Benchmark an LC-MS feature alignment method against a manually annotated feature dataset by computing TP/FP/TN/FN counts and the alignment metrics precision, recall, F1, F_ACC and A_ACC across benchmark datasets such as TripleTOF 6600, QE HF and MTBLS562.

## When to use

Use when you need to quantitatively evaluate how accurately a feature alignment method (e.g. G-Aligner, MZmine2, OpenMS, XCMS) aligns consensus features across multiple LC-MS runs against a manually annotated reference.

## When NOT to use

- You only have raw LC-MS data (e.g. mzML files) and still need feature extraction — evaluation requires already-extracted feature lists with m/z, RT and area columns
- You want to evaluate feature detection or quantification accuracy rather than alignment accuracy across runs
- Your dataset lacks manually annotated library analytes, so TP/FP/F_ACC/A_ACC cannot be computed

## Inputs

- Manually annotated feature dataset per benchmark (m/z, RT, area of library analytes) from Zenodo deposit 10.5281/zenodo.8313034
- Feature extraction results in csv format containing m/z, RT and area columns (e.g. *_results_metapro, *_results_mzmine2, *_results_openms, *_results_xcms directories)
- G-Aligner feature alignment outputs saved in the experiment folder

## Outputs

- TP/FP/TN/FN counts per alignment configuration
- Precision, recall, F1 per alignment configuration
- F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned) per benchmark dataset
- Comparison summary table mirroring Tables 1–3 structure (e.g. solver_comparison.csv with dataset, solver, F_ACC, A_ACC, running time)

## How to apply

Place the downloaded Zenodo deposit (doi:10.5281/zenodo.8313034) into the repository layout so that manually annotated feature lists and mzML-derived feature tables for TripleTOF_6600, QE_HF and MTBLS562 sit under data/ alongside the evaluation scripts. Run the alignment (e.g. G-Aligner via data/evaluate_metapro_galigner.py with solver-specific parameters from supplementary Tables S1–S3) so that alignment results land in the experiment folder. Match aligned consensus features against the manually annotated library analytes (m/z, RT, area of library analytes) to compute TP/FP/TN/FN counts, then derive precision, recall, F1, F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned). Aggregate the per-dataset and per-solver (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) metrics into a comparison table mirroring Table 1–3, and verify the relative ordering against reported results: combinatorial optimization solvers above MZmine2, OpenMS, XCMS and local bipartite matching, with up to 9.8% and 26.6% increases in accurately aligned features and analytes, and VLSNS up to 9.6 times faster than Gurobi/Greedy.

## Related tools

- **G-Aligner** (Produces the feature alignment results (consensus features) whose accuracy is evaluated; also benchmarked as the method under test with solvers Gurobi, Greedy, VLSNS_MSR and VLSNS_MSG) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison baseline whose built-in feature alignment accuracy is evaluated against manual annotation)
- **OpenMS** (Comparison baseline evaluated via evaluate_metapro_openms.py)
- **XCMS** (Comparison baseline evaluated via evaluate_metapro_xcms.Rmd)
- **Python** (Runs evaluation scripts (evaluate_metapro_galigner.py, evaluate_mzmine2_galigner.py, evaluate_openms_galigner.py, metapro_result_comparison.py) and aggregates metrics with pandas/numpy)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Computed F_ACC and A_ACC for G-Aligner on all three benchmark datasets should be the highest among MZmine2, OpenMS, XCMS and local bipartite matching, with reported increases of up to 9.8% (features) and 26.6% (analytes)
- VLSNS solver runtime should be up to 9.6 times faster than Gurobi and Greedy solvers at Gurobi-level accuracy (VLSNS with MSG)
- Greedy solver F_ACC/A_ACC should be below the global solvers (Gurobi, VLSNS)
- Per-dataset TP/FP/TN/FN counts must sum consistently with the manually annotated feature counts, and precision/recall/F1 must be derivable from those counts
- Alignment outputs must exist in the experiment folder after each run, and the aggregated table columns (dataset, solver, F_ACC, A_ACC, running time) must be populated for all solver–dataset combinations

## Limitations

- Main limitation of G-Aligner is lack of computing speed: the multidimensional assignment problem is NP-Hard and requires a lot of computing time; a gap remains between provided acceleration methods and painless application
- Optimized parameters for each dataset are reported only in supplementary tables (S1–S3), not in the main text
- GPU acceleration is not yet implemented
- Evaluation depends on the manually annotated feature dataset; accuracy claims are only as good as that annotation

## Evidence

- [methods] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [discussion] comparing alignment results with manually annotated reference using evaluation script computing TP/FP/TN/FN, precision, recall, F1, F_ACC and A_ACC: "G-Aligner achieved the highest feature alignment accuracy on all benchmark datasets with reasonable computational time."
- [abstract] up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [abstract] The VLSNS solver was up to 9.6 times faster than the Gurobi and Greedy solvers while empowering G-Aligner to achieve accurate feature alignment in acceptable time.: [evidence span withheld — anchor status: not_found]
- [readme] Feature alignment results are saved in ```experiment``` folder.: "Feature alignment results are saved in ```experiment``` folder."
