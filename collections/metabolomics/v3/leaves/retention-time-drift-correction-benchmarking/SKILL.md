---
name: retention-time-drift-correction-benchmarking
description: Use when evaluating or comparing LC–MS feature alignment algorithms (e.g.
  G-Aligner, MZmine2, OpenMS, XCMS, local bipartite matching) on datasets with nonlinear
  RT drift, especially when a manually annotated feature list is available as ground
  truth.
when_to_use_negative:
- Input is a single LC–MS run with no multi-run correspondence problem — the skill
  addresses aligning features across multiple related runs, not within-run peak picking.
- No manually annotated or library-analyte feature list is available and quantitative
  accuracy evaluation against ground truth is required — without the manual annotation
  data from Zenodo, the precision/recall/F_ACC/A_ACC evaluation cannot be computed.
- The task is de novo identification or annotation of metabolites rather than benchmarking
  the alignment step of an untargeted LC–MS metabolomics workflow.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: G-Aligner
  role: Primary feature alignment method under test; treats features and potential
    correspondences as a multipartite graph and solves multi-run feature matching
    as an unbalanced multidimensional assignment problem via three combinatorial optimization
    algorithms.
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison software whose alignment results (RT alignment with RANSAC plus
    pairwise nearest matching) are evaluated as a baseline against G-Aligner.
- name: OpenMS
  role: Comparison software whose clustering-based feature alignment is evaluated
    as a baseline against G-Aligner.
- name: XCMS
  role: Comparison software whose clustering-based feature alignment is evaluated
    as a baseline against G-Aligner.
- name: LWBmatch
  role: Baseline that formulates pairwise feature matching between runs as an unbalanced
    weighted bipartite matching problem.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:4bdfb79ce9aa2533b8a44820e1136ed827e1f62895a8bdbb26de9a3130712913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - LWBmatch
  - MZmine2
  - OpenMS
  - Python
  - XCMS
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- retention-time-drift-modelling
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# retention-time-drift-correction-benchmarking

## Summary

Benchmark feature alignment methods that must cope with nonlinear retention time (RT) drift between LC–MS runs, by evaluating them against manually annotated feature lists and reporting precision, recall, F1, F_ACC and A_ACC on benchmark datasets such as TripleTOF_6600, QE_HF and MTBLS562.

## When to use

Use when evaluating or comparing LC–MS feature alignment algorithms (e.g. G-Aligner, MZmine2, OpenMS, XCMS, local bipartite matching) on datasets with nonlinear RT drift, especially when a manually annotated feature list is available as ground truth.

## When NOT to use

- Input is a single LC–MS run with no multi-run correspondence problem — the skill addresses aligning features across multiple related runs, not within-run peak picking.
- No manually annotated or library-analyte feature list is available and quantitative accuracy evaluation against ground truth is required — without the manual annotation data from Zenodo, the precision/recall/F_ACC/A_ACC evaluation cannot be computed.
- The task is de novo identification or annotation of metabolites rather than benchmarking the alignment step of an untargeted LC–MS metabolomics workflow.

## Inputs

- mzML files of benchmark metabolomics datasets (TripleTOF_6600, QE_HF, MTBLS562) from Zenodo 10.5281/zenodo.8313034
- Manually annotated feature lists containing m/z, RT and area information of library analytes
- Feature extraction CSV results (m/z, RT, area columns) from G-Aligner/MetaPro, MZmine2, OpenMS and XCMS
- Evaluation scripts (evaluate_metapro_galigner.py, evaluate_metapro_mzmine2.java, evaluate_metapro_openms.py, evaluate_metapro_xcms.Rmd, evaluate_mzmine2_galigner.py, evaluate_openms_galigner.py, evaluate_xcms_galigner.py)

## Outputs

- Feature alignment results saved in the experiment folder for each method and solver configuration
- Comparison tables of TP/FP/TN/FN, precision, recall, F1, F_ACC and A_ACC for each alignment method (mirroring Tables 1–3)
- Result comparison summaries from data/metapro_result_comparison.py and data/software_result_comparison.py

## How to apply

Download the benchmark mzML files, manually annotated feature lists and pre-extracted feature tables (m/z, RT, area CSVs) for the three public datasets (TripleTOF_6600, QE_HF, MTBLS562) from Zenodo 10.5281/zenodo.8313034 and lay them out under G-Aligner's data directory (e.g. TripleTOF_6600 plus TripleTOF_6600_results_metapro/mzmine2/openms/xcms). Run each alignment method — for G-Aligner, select each solver (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) and use the optimized parameters from supplementary Table S1 for the TripleTOF 6600 dataset — and save alignment results in the experiment folder. Evaluate each alignment result against the manually annotated reference using the provided evaluation scripts, computing TP/FP/TN/FN counts, precision, recall, F1, F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned). Compile metrics for all methods into a single comparison table mirroring the structure of Table 1. Validation: compare the computed values against the paper's reported tables; G-Aligner is expected to achieve the highest feature alignment accuracy, with up to 9.8% and 26.6% increase in accurately aligned features and analytes.

## Related tools

- **G-Aligner** (Primary feature alignment method under test; treats features and potential correspondences as a multipartite graph and solves multi-run feature matching as an unbalanced multidimensional assignment problem via three combinatorial optimization algorithms.) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison software whose alignment results (RT alignment with RANSAC plus pairwise nearest matching) are evaluated as a baseline against G-Aligner.)
- **OpenMS** (Comparison software whose clustering-based feature alignment is evaluated as a baseline against G-Aligner.)
- **XCMS** (Comparison software whose clustering-based feature alignment is evaluated as a baseline against G-Aligner.)
- **LWBmatch** (Baseline that formulates pairwise feature matching between runs as an unbalanced weighted bipartite matching problem.)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Precision, recall and F1 computed against the manually annotated feature list match the values reported in Table 1 (TripleTOF 6600), Table 2 (QE HF) and Table 3 (MTBLS562).
- F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned) reproduce the paper's reported numbers; G-Aligner should show the highest F_ACC/A_ACC among compared methods.
- G-Aligner's improvement over comparison software in accurately aligned features reaches up to 9.8% and in accurately aligned analytes up to 26.6%.
- Integrating G-Aligner into MZmine2/OpenMS/XCMS workflows should improve each software's own alignment accuracy on its self-extracted features (software_result_comparison.py outputs).
- Alignment results are written to the experiment folder with the expected schema (feature correspondences across all runs), and benchmark scripts data/metapro_result_comparison.py and data/software_result_comparison.py run without error.

## Limitations

- The main limitation of G-Aligner is the lack of computing speed; the multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time.
- Although two acceleration methods keep the running time within an acceptable range, there was still a gap between painless application; GPU acceleration is proposed but not yet implemented.
- Optimized per-dataset parameters are only documented in supplementary Tables S1–S3, not in the main text.
- RT drift distributions of library analytes (e.g. on the TripleTOF 6600 and QE HF datasets) are nonlinear, so retention time alignment alone cannot accurately correct retention time of each feature — naive nearest neighbor matching is insufficient.

## Evidence

- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [intro] Features have nonlinear RT drifts: "features have nonlinear retention time (RT) drifts between runs due to unavoidable slight changes in chromatogram conditions"
- [intro] Most methods rely on naive nearest neighbor search: "Most existing methods rely heavily on retention time alignment and only use naive nearest neighbor search in feature matching."
- [intro] G-Aligner uses RANSAC and OBI-Warp: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [methods] Manually annotated feature dataset for three public benchmark datasets: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [methods] The mzML files can be downloaded at Zenodo: "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [discussion] Main limitation is lack of computing speed: "The main limitation of G-Aligner is the lack of computing speed."
- [discussion] G-Aligner achieved the highest alignment accuracy with reasonable computational time: "G-Aligner achieved the highest feature alignment accuracy on all benchmark datasets with reasonable computational time."
- [readme] Benchmark datasets and evaluation methods available at Zenodo: "Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)."
