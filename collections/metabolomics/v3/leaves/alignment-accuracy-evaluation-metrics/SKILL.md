---
name: alignment-accuracy-evaluation-metrics
description: Use when an LC–MS feature alignment output (consensus feature table)
  must be quantitatively scored against a manually annotated feature dataset, e.g.
  benchmarking G-Aligner vs OpenMS/MZmine2/XCMS on MTBLS562, QE_HF or TripleTOF_6600.
when_to_use_negative:
- Input is raw mzML data that has not yet been feature-detected and aligned — run
  feature extraction and alignment first before evaluation is possible
- No manually annotated ground-truth feature list exists for the dataset — F_ACC/A_ACC
  cannot be computed and only relative/indirect comparisons are possible
- The goal is runtime/speed benchmarking rather than alignment accuracy — the paper's
  own limitation is that G-Aligner's multidimensional assignment (NP-Hard) lacks computing
  speed, which these accuracy metrics do not capture
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Alignment method under evaluation; its aligned consensus features are scored
    for F_ACC/A_ACC against the manual annotation
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OpenMS
  role: Baseline alignment method whose native feature-grouping output is evaluated
    with evaluate_metapro_openms.py / evaluate_openms_galigner.py
- name: MZmine2
  role: Baseline alignment method evaluated via evaluate_metapro_mzmine2.java and
    evaluate_mzmine2_galigner.py
- name: XCMS
  role: Baseline alignment method evaluated via evaluate_metapro_xcms.Rmd and evaluate_xcms_galigner.py
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:bf9f3db8e95220d00e41e35837405e41c0ca3dc889a72674f3947d99a552082a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# alignment-accuracy-evaluation-metrics

## Summary

Evaluates LC–MS feature alignment quality by comparing aligned consensus features against manually annotated ground-truth feature lists (m/z, RT, area of library analytes), computing TP/FP/TN/FN, precision, recall, F1, feature alignment accuracy (F_ACC) and analyte alignment accuracy (A_ACC).

## When to use

Use when an LC–MS feature alignment output (consensus feature table) must be quantitatively scored against a manually annotated feature dataset, e.g. benchmarking G-Aligner vs OpenMS/MZmine2/XCMS on MTBLS562, QE_HF or TripleTOF_6600.

## When NOT to use

- Input is raw mzML data that has not yet been feature-detected and aligned — run feature extraction and alignment first before evaluation is possible
- No manually annotated ground-truth feature list exists for the dataset — F_ACC/A_ACC cannot be computed and only relative/indirect comparisons are possible
- The goal is runtime/speed benchmarking rather than alignment accuracy — the paper's own limitation is that G-Aligner's multidimensional assignment (NP-Hard) lacks computing speed, which these accuracy metrics do not capture

## Inputs

- Aligned consensus feature output (feature alignment results saved in the 'experiment' folder)
- Manually annotated feature lists with m/z, RT and area information of library analytes (from Zenodo 10.5281/zenodo.8313034)
- Deposited evaluation scripts (evaluate_metapro_galigner.py, evaluate_metapro_openms.py, evaluate_mzmine2_galigner.py, evaluate_xcms_galigner.py, metapro_result_comparison.py, software_result_comparison.py)
- Per-dataset optimized parameter settings from supplementary Tables S1–S3

## Outputs

- Metrics table (e.g. evaluation_results.csv) with TP, FP, TN, FN, precision, recall, F1, F_ACC and A_ACC per method/dataset
- Software/benchmark comparison tables from metapro_result_comparison.py and software_result_comparison.py

## How to apply

Download the benchmark datasets (mzML files), manually annotated feature lists (containing m/z, RT and area information of library analytes) and the deposited evaluation scripts from Zenodo deposit 10.5281/zenodo.8313034, and arrange them under ROOT_PATH/data (e.g. data/QE_HF, data/QE_HF_results_metapro) per the repository structure. Run the alignment method under test (e.g. G-Aligner via data/evaluate_metapro_galigner.py, with per-dataset optimized parameters from supplementary Tables S1–S3), then execute the matching evaluation script (evaluate_metapro_galigner.py, evaluate_openms_galigner.py, evaluate_mzmine2_galigner.py or evaluate_xcms_galigner.py) to compare the aligned consensus features against the ground truth, yielding TP, FP, TN, FN, precision, recall, F1, F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes). For G-Aligner, repeat across its three combinatorial optimization solvers to identify the highest-accuracy configuration, and compile all metrics into a single results table (e.g. evaluation_results.csv) mirroring the paper's Table 2. Acceptance is numeric agreement with the paper: e.g. on QE HF manually annotated features G-Aligner reached F1 0.998, F_ACC 0.997 and A_ACC 0.989.

## Related tools

- **G-Aligner** (Alignment method under evaluation; its aligned consensus features are scored for F_ACC/A_ACC against the manual annotation) — https://github.com/CSi-Studio/G-Aligner
- **OpenMS** (Baseline alignment method whose native feature-grouping output is evaluated with evaluate_metapro_openms.py / evaluate_openms_galigner.py)
- **MZmine2** (Baseline alignment method evaluated via evaluate_metapro_mzmine2.java and evaluate_mzmine2_galigner.py)
- **XCMS** (Baseline alignment method evaluated via evaluate_metapro_xcms.Rmd and evaluate_xcms_galigner.py)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Computed F1, F_ACC and A_ACC match the paper's reported values within reasonable tolerance (e.g. QE HF manual annotation: F1 0.998, feature accuracy 0.997, analyte accuracy 0.989)
- Integration experiments reproduce the reported gains over OpenMS's native alignment: ~2.4%, 1.5% and 1.7% F_ACC and 13.5%, 10.6% and 18.4% A_ACC on the TripleTOF 6600, QE HF and MTBLS562 datasets
- Results table exists with all required metric columns (TP, FP, TN, FN, precision, recall, F1, F_ACC, A_ACC) populated, mirroring the paper's Table 2
- Aligned consensus features are present in the 'experiment' folder after running the evaluation script
- All three G-Aligner combinatorial optimization solvers have been compared to confirm which achieves the highest accuracy

## Limitations

- Optimized per-dataset parameters are reported only in supplementary Tables S1–S3, not in the main text, so evaluation is sensitive to using the correct dataset-specific settings
- The evaluation depends on manually annotated feature lists, which exist only for the three public benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600) provided via Zenodo
- These metrics quantify accuracy only; they do not reflect G-Aligner's main limitation of computing speed (NP-Hard multidimensional assignment), where a gap remains between provided acceleration methods and painless application

## Evidence

- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [readme] Manual annotation dataset description: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes and can be used in evaluations of feature detection,"
- [readme] Evaluation materials availability: "Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)."
- [methods] Where alignment results are written: "Feature alignment results are saved in ```experiment``` folder."
