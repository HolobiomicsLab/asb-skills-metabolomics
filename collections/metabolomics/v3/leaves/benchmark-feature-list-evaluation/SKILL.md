---
name: benchmark-feature-list-evaluation
description: Use when you have feature lists (CSV with m/z, RT, area columns) extracted
  from LC–MS runs of the MTBLS562, QE_HF or TripleTOF_6600 benchmark datasets and
  need to score alignment accuracy against manually annotated library analytes.
when_to_use_negative:
- Your input is raw MS data (mzML) requiring feature detection/quantification rather
  than evaluation of already-extracted feature lists
- Your datasets are not among the benchmark collections (MTBLS562, QE_HF, TripleTOF_6600)
  and no manual annotation of library analytes exists to score against
- You need real-time or speed-critical alignment — the multidimensional assignment
  formulation is NP-Hard and the paper reports computing speed as its main limitation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: G-Aligner
  role: Graph-based alignment method evaluated on the benchmark feature lists; its
    results are compared against MZmine2, OpenMS and XCMS
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Baseline feature alignment software whose extracted features and alignment
    results are benchmarked
- name: OpenMS
  role: Baseline feature alignment software whose extracted features and alignment
    results are benchmarked
- name: XCMS
  role: Baseline feature alignment software whose extracted features and alignment
    results are benchmarked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:742c4120b777b60715369bd9d19a1d3a32fdcbc67f53530ffef58057078e4f7c
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# benchmark-feature-list-evaluation

## Summary

Evaluate feature alignment/detection outputs on the three manually annotated LC–MS benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600), comparing results from G-Aligner, MZmine2, OpenMS and XCMS against targeted annotations of library analytes containing m/z, RT and area information.

## When to use

Use when you have feature lists (CSV with m/z, RT, area columns) extracted from LC–MS runs of the MTBLS562, QE_HF or TripleTOF_6600 benchmark datasets and need to score alignment accuracy against manually annotated library analytes.

## When NOT to use

- Your input is raw MS data (mzML) requiring feature detection/quantification rather than evaluation of already-extracted feature lists
- Your datasets are not among the benchmark collections (MTBLS562, QE_HF, TripleTOF_6600) and no manual annotation of library analytes exists to score against
- You need real-time or speed-critical alignment — the multidimensional assignment formulation is NP-Hard and the paper reports computing speed as its main limitation

## Inputs

- Manually annotated feature lists (CSV with m/z, RT, area columns) for MTBLS562, QE_HF and TripleTOF_6600 from Zenodo doi:10.5281/zenodo.8313034
- Software-extracted untargeted feature lists from mzmine2, openms, xcms and metapro (*_results_* folders)
- mzML files of the metabolomics datasets from Zenodo doi:10.5281/zenodo.8313034
- Evaluation scripts: data/metapro_result_comparison.py, data/software_result_comparison.py, data/evaluate_metapro_galigner.py

## Outputs

- Alignment accuracy results (accurately aligned features and analytes) per method per dataset
- Consensus feature groups with m/z, RT, area, checked for absence of ambiguous matched groups
- Feature alignment results saved in the 'experiment' folder

## How to apply

Download the targeted annotation results, evaluation methods and mzML files from Zenodo (doi 10.5281/zenodo.8313034) and place them under data/ as MTBLS562, QE_HF and TripleTOF_6600 folders plus the *_results_metapro/mzmine2/openms/xccms folders. Feature lists must be CSV files containing m/z, RT and area columns, either the manual annotations of library analytes or software-extracted untargeted features. Run the benchmark driver scripts (data/metapro_result_comparison.py, data/software_result_comparison.py) to compare alignment methods, and per-method scripts (e.g. data/evaluate_metapro_galigner.py, adjusting parameters) to produce alignment results saved in the 'experiment' folder. Judge correctness by counting accurately aligned features and analytes relative to the manual annotations, checking for ambiguous matched groups in consensus features, and comparing against the reported baselines of MZmine2, OpenMS and XCMS.

## Related tools

- **G-Aligner** (Graph-based alignment method evaluated on the benchmark feature lists; its results are compared against MZmine2, OpenMS and XCMS) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Baseline feature alignment software whose extracted features and alignment results are benchmarked)
- **OpenMS** (Baseline feature alignment software whose extracted features and alignment results are benchmarked)
- **XCMS** (Baseline feature alignment software whose extracted features and alignment results are benchmarked)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Count of accurately aligned features and analytes relative to manual annotations should improve by up to 9.8% and 26.6% over existing methods when using G-Aligner
- Resulting consensus feature groups contain no ambiguous matched groups
- Benchmark scripts (data/metapro_result_comparison.py, data/software_result_comparison.py) run end-to-end on all three datasets without errors
- Alignment results are written to the 'experiment' folder in the expected output format
- Accuracy on manually annotated feature lists matches or exceeds results on software-extracted (mzmine2, openms, xcms) feature lists

## Limitations

- The main limitation of G-Aligner is the lack of computing speed; the multidimensional assignment problem is NP-Hard and requires a lot of computing time
- Optimized parameters used in evaluations are only reported in supplementary tables (S1–S3), not the main text
- Raw MS files must be obtained from a Google Drive link while mzML and annotations come from Zenodo, so data acquisition depends on external hosting

## Evidence

- [readme] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [methods] Feature alignment results are saved in experiment folder: "Feature alignment results are saved in ```experiment``` folder."
- [discussion] The main limitation of G-Aligner is the lack of computing speed: "The main limitation of G-Aligner is the lack of computing speed."
