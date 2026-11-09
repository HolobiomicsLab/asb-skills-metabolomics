---
name: lcms-feature-alignment-evaluation
description: Use when benchmarking feature alignment methods (e.g. G-Aligner, MZmine2,
  OpenMS, XCMS) on mzML-derived feature lists against a manually annotated ground
  truth of library analytes.
when_to_use_negative:
- Input is only raw MS data without a ground-truth annotation of library analytes
  — the evaluation requires manually annotated feature lists to compute F_ACC/A_ACC
- The goal is feature detection or quantification accuracy rather than multi-run alignment
  accuracy
- Datasets with fewer than two runs, where multi-run feature matching/alignment is
  not the task under test
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Feature alignment method under evaluation; treats features and potential correspondences
    as nodes and edges of a multipartite graph and solves multi-run matching as an
    unbalanced multidimensional assignment problem
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison software whose native alignment and self-extracted features provide
    the MZmine2 baseline for F_ACC/A_ACC gain computation
- name: OpenMS
  role: Comparison software evaluated against G-Aligner on the three benchmark datasets
- name: XCMS
  role: Comparison software; its native alignment results are the baseline for computing
    G-Aligner integration gains via data/evaluate_xcms_galigner.py
- name: RANSAC
  role: Retention time coarse registration between runs in the G-Aligner alignment
    stage prior to evaluation
- name: OBI-Warp
  role: Retention time coarse registration algorithm used with RANSAC in the G-Aligner
    alignment stage
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:0470723a94c336beb0378378b2fb133f07555b108ed92b5d78457fd7c6051a3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OBI-Warp
  - OpenMS
  - RANSAC
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lcms-feature-alignment-evaluation

## Summary

Evaluates multi-run LC-MS feature alignment accuracy by comparing aligned consensus features against manually annotated feature lists (m/z, RT, area of library analytes), computing F1, feature alignment accuracy (F_ACC) and analyte alignment accuracy (A_ACC) per dataset.

## When to use

Use when benchmarking feature alignment methods (e.g. G-Aligner, MZmine2, OpenMS, XCMS) on mzML-derived feature lists against a manually annotated ground truth of library analytes.

## When NOT to use

- Input is only raw MS data without a ground-truth annotation of library analytes — the evaluation requires manually annotated feature lists to compute F_ACC/A_ACC
- The goal is feature detection or quantification accuracy rather than multi-run alignment accuracy
- Datasets with fewer than two runs, where multi-run feature matching/alignment is not the task under test

## Inputs

- mzML raw LC-MS files for benchmark datasets (MTBLS562, QE HF, TripleTOF 6600) from Zenodo 10.5281/zenodo.8313034
- manually annotated feature lists containing m/z, RT and area of library analytes (ground truth)
- untargeted extracted feature lists in csv format with m/z, RT and area columns from MZmine2, OpenMS or XCMS
- G-Aligner alignment results written to the experiment folder

## Outputs

- Per-method metrics table: TP, FP, TN, FN, precision, recall, F1 score, feature alignment accuracy (F_ACC), analyte alignment accuracy (A_ACC), computational time (e.g. alignment_accuracy_mtbls562.csv, evaluation_results.csv)
- F_ACC/A_ACC improvement deltas over native MZmine2/XCMS/OpenMS alignment (cf. Tables 4-6)

## How to apply

Download the benchmark mzML files and manually annotated feature lists from Zenodo (10.5281/zenodo.8313034) and arrange them under the G-Aligner project tree (e.g. data/MTBLS562, data/QE_HF, data/TripleTOF_6600). Run the alignment tool (G-Aligner builds a multipartite graph over features of all runs, formulates multi-run matching as an unbalanced multidimensional assignment problem solved by three combinatorial optimization algorithms, after RANSAC/OBI-Warp RT coarse registration), then execute the evaluation script (e.g. data/evaluate_metapro_galigner.py, data/evaluate_mzmine2_galigner.py) configured with the optimized parameters from supplementary Tables S1-S3. Match aligned features against the manual annotation to count accurately aligned features and analytes, computing TP/FP/TN/FN, precision, recall, F1, F_ACC and A_ACC. For integration benchmarks (e.g. G-Aligner on MZmine2 or XCMS self-extracted features), compare F_ACC/A_ACC gains over the software's native alignment using data/software_result_comparison.py.

## Related tools

- **G-Aligner** (Feature alignment method under evaluation; treats features and potential correspondences as nodes and edges of a multipartite graph and solves multi-run matching as an unbalanced multidimensional assignment problem) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison software whose native alignment and self-extracted features provide the MZmine2 baseline for F_ACC/A_ACC gain computation)
- **OpenMS** (Comparison software evaluated against G-Aligner on the three benchmark datasets)
- **XCMS** (Comparison software; its native alignment results are the baseline for computing G-Aligner integration gains via data/evaluate_xcms_galigner.py)
- **RANSAC** (Retention time coarse registration between runs in the G-Aligner alignment stage prior to evaluation)
- **OBI-Warp** (Retention time coarse registration algorithm used with RANSAC in the G-Aligner alignment stage)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Reproduced F_ACC and A_ACC match the published G-Aligner rows (e.g. F1 1.000, feature accuracy 0.999, analyte accuracy 0.981 on MTBLS562 manually annotated features)
- Integration gains match Tables 4-6: F_ACC +1.7%/+9.0%/+0.5% and A_ACC +5.8%/+26.6%/+7.7% over MZmine2 on TripleTOF 6600/QE HF/MTBLS562
- For XCMS integration: F_ACC gains of 9.8%/2.6%/9.4% and A_ACC gains of 24.5%/11.4%/24.7%; flag any dataset deviating from these values
- Output CSV exists with all required metric columns (TP, FP, TN, FN, precision, recall, F1, F_ACC, A_ACC) populated
- Alignment results are written to the experiment folder after running the evaluation script from ROOT_PATH

## Limitations

- G-Aligner's main limitation is lack of computing speed; the multidimensional assignment problem is NP-Hard and requires substantial computing time despite two acceleration methods
- Optimized evaluation parameters are reported only in supplementary tables S1-S3 (e.g. Table S3 for MTBLS562), not in the main text
- GPU acceleration is not yet implemented, so a gap remains between the provided acceleration methods and painless application

## Evidence

- [abstract] Best performance claim: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [readme] Manual annotation dataset description: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [intro] Algorithm formulation: "G-Aligner formats the potential correspondences between all features as a multipartite graph, and solves the multi-run feature matching problem as an unbalanced multidimensional assignment problem"
- [intro] RT alignment stage: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [methods] Output location: "Feature alignment results are saved in ```experiment``` folder."
- [discussion] Speed limitation: "The main limitation of G-Aligner is the lack of computing speed."
