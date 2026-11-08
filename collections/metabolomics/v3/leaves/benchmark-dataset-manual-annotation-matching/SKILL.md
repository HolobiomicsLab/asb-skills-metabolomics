---
name: benchmark-dataset-manual-annotation-matching
description: Use when you need to quantify feature alignment accuracy of an alignment
  tool (G-Aligner, MZmine2, OpenMS, XCMS, LWBmatch) against a ground-truth manually
  annotated feature list on public metabolomics benchmark data.
when_to_use_negative:
- Input is raw mzML/LC-MS data with no extracted feature list — feature detection/quantification
  must be performed first before alignment matching can be evaluated
- No manually annotated ground-truth feature list exists for the dataset — F_ACC/A_ACC
  cannot be computed without the reference annotation
- Goal is retention time drift correction itself rather than scoring alignment accuracy
  against a reference
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: G-Aligner
  role: Alignment method under evaluation; performs RANSAC/OBI-Warp RT coarse registration
    and multipartite-graph feature matching, results saved to the experiment folder
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Baseline alignment method whose self-extracted feature lists and alignment
    results are scored against manual annotations
- name: OpenMS
  role: Baseline clustering-based alignment method compared in the benchmark evaluation
- name: XCMS
  role: Baseline clustering-based alignment method compared in the benchmark evaluation
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:29fad5d00f525adc29c45464a05c4f8ff921cbee0cff23851458aff9daa8ac7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# benchmark-dataset-manual-annotation-matching

## Summary

Evaluate LC-MS feature alignment methods by comparing their aligned consensus feature lists against manually annotated reference feature lists (m/z, RT, area of library analytes) on the TripleTOF 6600, QE HF and MTBLS562 benchmark datasets, computing TP/FP/TN/FN, precision, recall, F1, F_ACC and A_ACC.

## When to use

Use when you need to quantify feature alignment accuracy of an alignment tool (G-Aligner, MZmine2, OpenMS, XCMS, LWBmatch) against a ground-truth manually annotated feature list on public metabolomics benchmark data.

## When NOT to use

- Input is raw mzML/LC-MS data with no extracted feature list — feature detection/quantification must be performed first before alignment matching can be evaluated
- No manually annotated ground-truth feature list exists for the dataset — F_ACC/A_ACC cannot be computed without the reference annotation
- Goal is retention time drift correction itself rather than scoring alignment accuracy against a reference

## Inputs

- Manually annotated feature lists (m/z, RT, area of library analytes) for TripleTOF 6600, QE HF and MTBLS562 datasets from Zenodo deposit 10.5281/zenodo.8313034
- Feature extraction results in csv format containing m/z, RT and area columns (untargeted self-extracted features from MZmine2, OpenMS, XCMS or MetaPro)
- Alignment tool outputs (consensus feature lists) saved in the experiment folder

## Outputs

- Per-dataset evaluation metrics: TP/FP/TN/FN counts, precision, recall, F1, F_ACC, A_ACC
- Comparison summary tables mirroring Tables 1-6 of the paper (e.g. data/software_result_comparison.py output)

## How to apply

Download the manually annotated feature lists and mzML-derived feature extraction results for the TripleTOF 6600, QE_HF and MTBLS562 datasets from Zenodo (doi 10.5281/zenodo.8313034) and arrange them under the G-Aligner project tree (data/TripleTOF_6600, QE_HF, MTBLS562). Run the alignment method with the optimized parameters from supplementary Tables S1-S3, then evaluate each alignment result against the manual annotation using the provided evaluation scripts (e.g. data/evaluate_metapro_galigner.py), computing TP/FP/TN/FN counts, precision, recall, F1, F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned). For software self-extracted features, consume the csv feature lists (m/z, RT, area columns) and apply G-Aligner's RANSAC/OBI-Warp RT coarse registration plus multipartite-graph matching before evaluation. Compile per-dataset metric tables mirroring Tables 1-6 and validate gains against the reported values (e.g. F_ACC +1.7%, +9.0%, +0.5%; A_ACC +5.8%, +26.6%, +7.7% over MZmine2 on self-extracted features).

## Related tools

- **G-Aligner** (Alignment method under evaluation; performs RANSAC/OBI-Warp RT coarse registration and multipartite-graph feature matching, results saved to the experiment folder) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Baseline alignment method whose self-extracted feature lists and alignment results are scored against manual annotations)
- **OpenMS** (Baseline clustering-based alignment method compared in the benchmark evaluation)
- **XCMS** (Baseline clustering-based alignment method compared in the benchmark evaluation)

## Examples

```
python data/evaluate_mzmine2_galigner.py  # after placing Zenodo data under data/{TripleTOF_6600,QE_HF,MTBLS562}; results written to experiment folder
```

## Evaluation signals

- Computed F_ACC/A_ACC gains over MZmine2 on self-extracted features match Tables 4-6: F_ACC +1.7% (TripleTOF 6600), +9.0% (QE HF), +0.5% (MTBLS562); A_ACC +5.8%, +26.6%, +7.7%
- G-Aligner solvers (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) achieve the highest precision/recall/F1/F_ACC/A_ACC among all compared methods on the manually annotated TripleTOF 6600 data, mirroring Table 1
- F_ACC is the percentage of accurately aligned features and A_ACC the percentage of analytes with all replicates accurately aligned — both computable only when every replicate's alignment matches the manual annotation
- Alignment results are written to the experiment folder in the expected project layout and are consumed by data/evaluate_metapro_galigner.py and data/metapro_result_comparison.py without path errors

## Limitations

- Optimized evaluation parameters are reported only in supplementary Tables S1-S3, not in the main text
- G-Aligner's main limitation is lack of computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computation time; GPU acceleration is not yet implemented
- The manually annotated feature dataset covers only three public benchmark datasets (TripleTOF 6600, QE HF, MTBLS562); annotations contain m/z, RT, area of library analytes

## Evidence

- [readme] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes and can be used in evaluations of feature detection, quantification and alignment accuracy.: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [methods] computing TP/FP/TN/FN counts, precision, recall, F1, F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned): [evidence span withheld — anchor status: not_found]
- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [intro] G-Aligner uses the existing RANSAC and OBI-Warp algorithms to perform stable coarse registration between runs: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [discussion] Table S1, Optimized parameters used in the TripleTOF 6600 dataset evaluation: "Table S1, Optimized parameters used in the TripleTOF 6600 dataset evaluation"
