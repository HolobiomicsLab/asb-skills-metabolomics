---
name: feature-matching-accuracy-evaluation
description: Use when quantifying how accurately a feature alignment method matched
  features across multiple LC-MS runs, by comparing aligned results against manually
  annotated library analytes (m/z, RT, area) with F_ACC/A_ACC metrics.
when_to_use_negative:
- The input is raw LC-MS data without prior feature extraction — alignment evaluation
  requires csv feature lists with m/z, RT and area columns
- The dataset lacks a manually annotated set of library analytes, so F_ACC/A_ACC cannot
  be computed against ground truth
- The goal is to evaluate feature detection or quantification accuracy rather than
  feature alignment/matching accuracy
tools:
- name: G-Aligner
  role: Alignment method whose matched features are scored for F_ACC/A_ACC against
    manually annotated library analytes
  repo: https://github.com/CSi-Studio/G-Aligner
- name: XCMS
  role: Comparison software whose native alignment results and self-extracted features
    provide the baseline F_ACC/A_ACC for improvement measurement
- name: MZmine2
  role: Comparison software providing self-extracted feature lists and native alignment
    baseline for accuracy comparison
- name: OpenMS
  role: Comparison software providing self-extracted feature lists and native alignment
    baseline for accuracy comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:775ffc00cc906cd999cf2eab8650a3bd6bbfa2a8608aca5f61f5e5230c95e751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# feature-matching-accuracy-evaluation

## Summary

Evaluates the accuracy of LC-MS feature alignment/matching by scoring aligned consensus features against manually annotated library analytes, using feature accuracy (F_ACC) and analyte accuracy (A_ACC) metrics on the TripleTOF 6600, QE HF and MTBLS562 benchmark datasets.

## When to use

Use when quantifying how accurately a feature alignment method matched features across multiple LC-MS runs, by comparing aligned results against manually annotated library analytes (m/z, RT, area) with F_ACC/A_ACC metrics.

## When NOT to use

- The input is raw LC-MS data without prior feature extraction — alignment evaluation requires csv feature lists with m/z, RT and area columns
- The dataset lacks a manually annotated set of library analytes, so F_ACC/A_ACC cannot be computed against ground truth
- The goal is to evaluate feature detection or quantification accuracy rather than feature alignment/matching accuracy

## Inputs

- mzML files of the metabolomics benchmark datasets (from Zenodo 10.5281/zenodo.8313034)
- Software-extracted untargeted feature lists in csv format containing m/z, RT and area columns (e.g. from XCMS, MZmine2, OpenMS)
- Manually annotated feature dataset containing m/z, RT and area information of library analytes for the three benchmark datasets
- Aligned feature-alignment results (e.g. G-Aligner output, or native XCMS/MZmine2/OpenMS alignment results)

## Outputs

- Per-dataset F_ACC (feature accuracy) and A_ACC (analyte accuracy) values for each alignment method
- Comparison of alignment accuracies between G-Aligner and native software methods across TripleTOF 6600, QE HF and MTBLS562 (as in Tables 4, 5 and 6)
- Feature alignment results saved in the experiment folder

## How to apply

Download the mzML files, software feature-extraction results and manual annotation results from Zenodo 10.5281/zenodo.8313034, then organize them under data/TripleTOF_6600_results_xcms, data/QE_HF_results_xcms and data/MTBLS562_results_xcms (analogous folders exist for MZmine2, OpenMS and MetaPro). The manual annotation dataset contains m/z, RT and area information of library analytes for the three public benchmark datasets. Run the dataset-specific evaluation script (e.g. data/evaluate_xcms_galigner.py) to compute F_ACC (fraction of accurately aligned features) and A_ACC (fraction of accurately aligned analytes) of the aligned feature lists against the manually annotated analytes; alignment results are written to the experiment folder. Then run data/software_result_comparison.py to benchmark the alignment method against the native alignment methods of the comparison software (XCMS, MZmine2, OpenMS). Judge success by whether the computed F_ACC/A_ACC gains reproduce the reported values (F_ACC +9.8%, +2.6%, +9.4%; A_ACC +24.5%, +11.4%, +24.7% for TripleTOF 6600, QE HF and MTBLS562 respectively when G-Aligner is integrated into XCMS), flagging any dataset that deviates.

## Related tools

- **G-Aligner** (Alignment method whose matched features are scored for F_ACC/A_ACC against manually annotated library analytes) — https://github.com/CSi-Studio/G-Aligner
- **XCMS** (Comparison software whose native alignment results and self-extracted features provide the baseline F_ACC/A_ACC for improvement measurement)
- **MZmine2** (Comparison software providing self-extracted feature lists and native alignment baseline for accuracy comparison)
- **OpenMS** (Comparison software providing self-extracted feature lists and native alignment baseline for accuracy comparison)

## Examples

```
python data/evaluate_xcms_galigner.py ; python data/software_result_comparison.py
```

## Evaluation signals

- Computed F_ACC gains over native XCMS match the reported +9.8% (TripleTOF 6600), +2.6% (QE HF) and +9.4% (MTBLS562)
- Computed A_ACC gains over native XCMS match the reported +24.5% (TripleTOF 6600), +11.4% (QE HF) and +24.7% (MTBLS562)
- Reproduced values agree with Tables 4, 5 and 6 of the paper; flag any dataset whose gains deviate
- Data directory structure matches the README layout (data/<dataset>_results_<software> folders with evaluation scripts present)
- Alignment results are written to the experiment folder as expected by the evaluation scripts

## Limitations

- The main limitation of G-Aligner is the lack of computing speed: the multidimensional assignment problem is NP-Hard and requires a lot of computing time; two acceleration methods exist but there is still a gap between painless application
- GPU acceleration to speed up parallel calculations is proposed but not yet implemented
- Optimized parameters used in the dataset evaluations are only reported in supplementary Tables S1–S3, not in the main text
- Ground-truth manual annotation covers only the three public benchmark datasets (TripleTOF 6600, QE_HF, MTBLS562); F_ACC/A_ACC cannot be computed for datasets without annotated library analytes

## Evidence

- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [methods] manual annotation of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [methods] evaluation outputs location: "Feature alignment results are saved in ```experiment``` folder."
- [readme] feature list format required for evaluation: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [abstract] integration into software workflows improves accuracy: "helped all comparison software obtain more accurate results on their self-extracted features by integrating G-Aligner to their analysis workflow"
