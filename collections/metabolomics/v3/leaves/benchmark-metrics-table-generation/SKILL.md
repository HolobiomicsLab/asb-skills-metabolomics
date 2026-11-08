---
name: benchmark-metrics-table-generation
description: Use when a graph-based alignment method (G-Aligner) has been run alongside
  a baseline workflow (OpenMS/MZmine2/XCMS) on self-extracted features and you must
  quantify alignment gains in a reproducible metrics table.
when_to_use_negative:
- The features are manually annotated library-analyte features rather than software
  self-extracted untargeted features — the 2.4%/1.5%/1.7% and 13.5%/10.6%/18.4% gains
  apply specifically to the OpenMS self-extracted-feature comparison
- No manually annotated ground-truth feature dataset exists for the data, since F_ACC
  and A_ACC can only be computed against the deposited manual annotations
- The task is detection or quantification evaluation rather than feature alignment
  accuracy — although the same manual annotation dataset can serve those evaluations,
  the metrics table here is specific to alignment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: G-Aligner
  role: Graph-based alignment method whose F_ACC/A_ACC are compared against the OpenMS
    baseline; installed via third_party/setup.py and requirements.txt, run through
    the deposited evaluation scripts
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OpenMS
  role: 'Baseline workflow: its feature-finding modules extract untargeted features
    and its native feature-grouping module provides the reference alignment to beat'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:ec535bc76c7b9ba3d762e20552bf0a802f8b01d2c6892cca1ba6f90c74382918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - OpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# benchmark-metrics-table-generation

## Summary

Generates a comparison metrics table quantifying how much G-Aligner improves feature alignment over a baseline tool's native method (e.g. OpenMS) on self-extracted untargeted features, using F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes) computed against manually annotated feature datasets.

## When to use

Use when a graph-based alignment method (G-Aligner) has been run alongside a baseline workflow (OpenMS/MZmine2/XCMS) on self-extracted features and you must quantify alignment gains in a reproducible metrics table.

## When NOT to use

- The features are manually annotated library-analyte features rather than software self-extracted untargeted features — the 2.4%/1.5%/1.7% and 13.5%/10.6%/18.4% gains apply specifically to the OpenMS self-extracted-feature comparison
- No manually annotated ground-truth feature dataset exists for the data, since F_ACC and A_ACC can only be computed against the deposited manual annotations
- The task is detection or quantification evaluation rather than feature alignment accuracy — although the same manual annotation dataset can serve those evaluations, the metrics table here is specific to alignment

## Inputs

- mzML files of the MTBLS562, QE_HF and TripleTOF_6600 datasets (Zenodo 10.5281/zenodo.8313034)
- Per-run feature lists (csv) self-extracted with OpenMS feature-finding modules, with m/z, RT and intensity/area columns
- Baseline alignment results from OpenMS's native feature-grouping module
- G-Aligner alignment results saved in the 'experiment' folder
- Manually annotated feature dataset (m/z, RT, area of library analytes) from Zenodo
- Deposited evaluation scripts (evaluate_openms_galigner.py, software_result_comparison.py)

## Outputs

- Comparison metrics table with F_ACC and A_ACC for baseline (OpenMS) and G-Aligner per dataset
- Computed accuracy gains (percentage-point deltas) of G-Aligner over the OpenMS native method
- Validation verdict that computed gains match reported values (~2.4%/1.5%/1.7% F_ACC; 13.5%/10.6%/18.4% A_ACC) within tolerance

## How to apply

Download mzML files, manual annotations, and deposited evaluation scripts (e.g. evaluate_openms_galigner.py, software_result_comparison.py) from Zenodo record 10.5281/zenodo.8313034. Align the per-run feature lists (csv with m/z, RT and area columns) with the baseline tool's native grouping module (OpenMS's feature-grouping), then re-align the same lists with G-Aligner using the per-dataset optimized parameters from supplementary Tables S1-S3 (TripleTOF_6600, QE_HF, MTBLS562). Run the deposited evaluation script for the workflow being tested to compute F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes) for both alignments against the manually annotated feature dataset. Assemble both methods' F_ACC/A_ACC values into one comparison table and verify the reported deltas (G-Aligner integrated into OpenMS: +2.4%, +1.5%, +1.7% feature accuracy and +13.5%, +10.6%, +18.4% analyte accuracy on TripleTOF 6600, QE HF and MTBLS562 respectively) match within reasonable tolerance; numeric agreement should be confirmed by expert review.

## Related tools

- **G-Aligner** (Graph-based alignment method whose F_ACC/A_ACC are compared against the OpenMS baseline; installed via third_party/setup.py and requirements.txt, run through the deposited evaluation scripts) — https://github.com/CSi-Studio/G-Aligner
- **OpenMS** (Baseline workflow: its feature-finding modules extract untargeted features and its native feature-grouping module provides the reference alignment to beat)

## Examples

```
cd ROOT_PATH && python data/software_result_comparison.py && python data/evaluate_openms_galigner.py
```

## Evaluation signals

- The comparison metrics table is produced for all three datasets (MTBLS562, QE_HF, TripleTOF_6600) with both F_ACC and A_ACC columns
- Computed F_ACC gains over the OpenMS baseline are approximately 2.4% (TripleTOF 6600), 1.5% (QE HF) and 1.7% (MTBLS562) within a reasonable tolerance
- Computed A_ACC gains over the OpenMS baseline are approximately 13.5%, 10.6% and 18.4% on TripleTOF 6600, QE HF and MTBLS562 respectively (expert_review on numeric agreement)
- G-Aligner alignment results were written to the 'experiment' folder and the evaluation was run with the per-dataset optimized parameters from supplementary Tables S1-S3
- Both alignments were scored against the same manually annotated feature dataset from Zenodo, so the two methods are directly comparable

## Limitations

- Optimized parameters are reported only in supplementary tables (S1-S3), not in the main text, so the evaluation depends on those tables
- G-Aligner's main limitation is computing speed: the multidimensional assignment problem is NP-Hard and, despite two acceleration methods, there is still a gap to painless application; GPU acceleration is not yet implemented
- Reported gains apply to the benchmark datasets evaluated; results depend on the deposited manual annotations and the specific OpenMS workflow used for feature extraction

## Evidence

- [abstract] OpenMS workflow gained 2.4%, 1.5% and 1.7% higher feature accuracy and 13.5%, 10.6% and 18.4% higher analyte accuracy: [evidence span withheld — anchor status: not_found]
- [readme] evaluation scripts from README: "│   ├── evaluate_openms_galigner.py
│   ├── evaluate_xcms_galigner.py
│   ├── metapro_result_comparison.py
│   ├── software_result_comparison.py"
- [readme] manual annotation dataset description from README: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes and can be used in evaluations of feature detection,"
- [methods] results folder from README: "Feature alignment results are saved in ```experiment``` folder."
- [discussion] computing speed limitation: "The main limitation of G-Aligner is the lack of computing speed."
