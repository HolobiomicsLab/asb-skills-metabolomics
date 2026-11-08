---
name: cross-dataset-algorithm-comparison
description: Use when a new alignment or feature-correspondence method must be validated
  against established software on ≥2 benchmark datasets with ground-truth manual annotations,
  and per-dataset accuracy deltas are the target output.
when_to_use_negative:
- The task is method development on a single dataset without manually annotated library
  analytes — no ground truth exists to compute F_ACC/A_ACC deltas
- The input is raw MS data rather than already-extracted feature lists; feature extraction,
  not cross-software alignment comparison, is the bottleneck
- Only one instrument/platform is in scope and the goal is a quick sanity check rather
  than cross-dataset robustness validation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Candidate alignment method evaluated on self-extracted features of each comparison
    software across all three datasets
  repo: https://github.com/CSi-Studio/G-Aligner
- name: XCMS
  role: Baseline software whose native alignment results are compared against G-Aligner-integrated
    results on its own extracted features
- name: MZmine2
  role: Baseline alignment method (RANSAC RT alignment plus reference-run pairwise
    nearest matching) included in the cross-software comparison
- name: OpenMS
  role: Baseline alignment method (nearest-feature clustering) included in the cross-software
    comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:9ffeac91f3c1a60c5fe9f861f027c3512745de37b3fadf056efd3c052f18bdc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-dataset-algorithm-comparison

## Summary

Benchmarks a feature alignment algorithm against native alignment methods (XCMS, MZmine2, OpenMS) across multiple public LC-MS metabolomics benchmark datasets, computing feature accuracy (F_ACC) and analyte accuracy (A_ACC) against manually annotated library analytes per dataset.

## When to use

Use when a new alignment or feature-correspondence method must be validated against established software on ≥2 benchmark datasets with ground-truth manual annotations, and per-dataset accuracy deltas are the target output.

## When NOT to use

- The task is method development on a single dataset without manually annotated library analytes — no ground truth exists to compute F_ACC/A_ACC deltas
- The input is raw MS data rather than already-extracted feature lists; feature extraction, not cross-software alignment comparison, is the bottleneck
- Only one instrument/platform is in scope and the goal is a quick sanity check rather than cross-dataset robustness validation

## Inputs

- mzML files of the three benchmark datasets (from Zenodo 10.5281/zenodo.8313034)
- Self-extracted feature lists in CSV format with m/z, RT and area columns (e.g. data/TripleTOF_6600_results_xcms, data/QE_HF_results_xcms, data/MTBLS562_results_xcms)
- Manual annotation results containing m/z, RT, area information of library analytes (ground truth)
- Dataset-specific optimized algorithm parameters (supplementary Tables S1–S3)
- Evaluation and comparison scripts (evaluate_*_galigner.py, software_result_comparison.py)

## Outputs

- Per-dataset F_ACC and A_ACC for the candidate algorithm on self-extracted features
- Per-dataset F_ACC and A_ACC for the native baseline method
- Per-dataset improvement deltas over the native method (e.g. +9.8% F_ACC, +24.5% A_ACC on TripleTOF 6600)
- Feature alignment results saved in the experiment folder
- Cross-dataset comparison table mirroring Tables 4, 5 and 6

## How to apply

Download mzML files, software-extracted feature lists and manual annotation results from Zenodo (10.5281/zenodo.8313034) and organize them per dataset (TripleTOF_6600, QE_HF, MTBLS562) under the repo's data/ layout. Run the candidate algorithm on each dataset's self-extracted feature lists using dataset-specific optimized parameters (reported in supplementary Tables S1–S3), then run the dataset-matched evaluation script (e.g. evaluate_xcms_galigner.py) to compute F_ACC and A_ACC against the manually annotated library analytes (m/z, RT, area). Run software_result_comparison.py to compare against the native alignment results of the same software for all datasets, then record per-dataset improvements and flag any dataset whose gain deviates from the reported values (e.g. G-Aligner-on-XCMS: F_ACC +9.8%, +2.6%, +9.4%; A_ACC +24.5%, +11.4%, +24.7% for TripleTOF 6600, QE HF and MTBLS562 respectively). The rationale is that a method only counts as robust if it wins consistently across datasets with different instruments (TripleTOF 6600, QE HF) and study designs (MTBLS562), not on a single dataset.

## Related tools

- **G-Aligner** (Candidate alignment method evaluated on self-extracted features of each comparison software across all three datasets) — https://github.com/CSi-Studio/G-Aligner
- **XCMS** (Baseline software whose native alignment results are compared against G-Aligner-integrated results on its own extracted features)
- **MZmine2** (Baseline alignment method (RANSAC RT alignment plus reference-run pairwise nearest matching) included in the cross-software comparison)
- **OpenMS** (Baseline alignment method (nearest-feature clustering) included in the cross-software comparison)

## Examples

```
python data/evaluate_xcms_galigner.py && python data/software_result_comparison.py
```

## Evaluation signals

- Computed F_ACC/A_ACC improvements per dataset reproduce the reported values (F_ACC +9.8%, +2.6%, +9.4%; A_ACC +24.5%, +11.4%, +24.7% for TripleTOF 6600, QE HF, MTBLS562); flag any deviating dataset
- The candidate method ranks best on all three datasets, not just one, matching the claim 'achieved the best feature alignment performance on all the three datasets'
- software_result_comparison.py runs to completion for all three datasets and produces per-software comparison tables
- Input feature lists contain m/z, RT and area columns as required by the supported CSV format
- Alignment outputs are written to the experiment folder for every dataset evaluated

## Limitations

- G-Aligner's main limitation is computing speed: the multidimensional assignment formulation is NP-Hard and requires substantial runtime even with the two provided acceleration methods
- Optimized parameters are only reported in supplementary tables (S1–S3), so per-dataset tuning must be sourced from the SI
- GPU acceleration is not yet implemented, so very large feature sets may remain slow

## Evidence

- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [abstract] Benchmark datasets, manual annotation results, evaluation methods and results are available: "Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034"
- [readme] README data layout for per-dataset comparison scripts: "```python data/software_result_comparison.py```"
- [readme] README feature-list input format requirement: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [intro] Comparison software baseline scope: "In comparison with feature alignment methods in OpenMS, MZmine2 and XCMS, G-Aligner showed significant improvements in feature alignment accuracy"
