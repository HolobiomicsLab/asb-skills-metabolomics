---
name: zenodo-benchmark-dataset-download
description: Use when a task requires the raw LC–MS benchmark data (MTBLS562, QE_HF,
  TripleTOF_6600), manual analyte annotations, or the deposited evaluation scripts
  needed to reproduce G-Aligner alignment-accuracy comparisons.
when_to_use_negative:
- Input is already a set of extracted feature CSVs with m/z, RT and area columns —
  G-Aligner consumes those directly and the mzML download is only needed to redo feature
  extraction
- The analysis targets a dataset outside MTBLS562, QE_HF or TripleTOF_6600 — this
  record only contains the paper's three public benchmark datasets
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: G-Aligner
  role: Benchmark repository whose README defines the data/ directory layout and demo
    scripts the downloaded Zenodo data must populate
  repo: https://github.com/CSi-Studio/G-Aligner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:d3f39026d7bcec471ec98bffe17407ae8bc7d0bfb30307754dbbf4b980bedc28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# zenodo-benchmark-dataset-download

## Summary

Downloads the G-Aligner paper's three metabolomics benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600) as mzML files, plus manually annotated feature lists and evaluation scripts, from Zenodo record 10.5281/zenodo.8313034 to enable reproduction of feature-alignment benchmarks.

## When to use

Use when a task requires the raw LC–MS benchmark data (MTBLS562, QE_HF, TripleTOF_6600), manual analyte annotations, or the deposited evaluation scripts needed to reproduce G-Aligner alignment-accuracy comparisons.

## When NOT to use

- Input is already a set of extracted feature CSVs with m/z, RT and area columns — G-Aligner consumes those directly and the mzML download is only needed to redo feature extraction
- The analysis targets a dataset outside MTBLS562, QE_HF or TripleTOF_6600 — this record only contains the paper's three public benchmark datasets

## Inputs

- Zenodo record 10.5281/zenodo.8313034 (benchmark mzML files, manual annotations, evaluation scripts)
- Google Drive folder with raw MS files (README mirror)
- G-Aligner repository layout (data/ directory conventions)

## Outputs

- mzML files for MTBLS562, QE_HF and TripleTOF_6600 runs
- Manually annotated feature dataset (m/z, RT, area of library analytes)
- Evaluation scripts (e.g. evaluate_openms_galigner.py, software_result_comparison.py) placed under data/ per README structure

## How to apply

Retrieve the mzML files of the MTBLS562, QE_HF and TripleTOF_6600 metabolomics datasets from Zenodo DOI 10.5281/zenodo.8313034; the same record also provides the manually annotated feature dataset (m/z, RT, area of library analytes), targeted annotation results, and evaluation scripts such as evaluate_openms_galigner.py. Place the downloaded data under the repo's data/ directory following the README layout (e.g. data/MTBLS562, data/QE_HF, data/TripleTOF_6600 alongside *_results_openms folders and evaluate_*.py scripts) so the benchmark comparison scripts resolve the paths. If raw vendor files are preferred, the README also links a Google Drive mirror of the raw MS files. This acquisition step precedes feature extraction with OpenMS/MZmine2/XCMS and any G-Aligner re-alignment run.

## Related tools

- **G-Aligner** (Benchmark repository whose README defines the data/ directory layout and demo scripts the downloaded Zenodo data must populate) — https://github.com/CSi-Studio/G-Aligner

## Evaluation signals

- data/ contains MTBLS562, QE_HF, TripleTOF_6600 folders plus their *_results_openms, *_results_mzmine2, *_results_xcms, *_results_metapro siblings as in the README tree
- Downloaded evaluation scripts (evaluate_openms_galigner.py, software_result_comparison.py, etc.) match the file list in the README
- Manually annotated feature files contain m/z, RT and area columns for library analytes
- mzML files load in OpenMS feature-finding modules without format errors
- Feature lists extracted from the mzML files can be aligned against the manual annotations to compute F_ACC and A_ACC matching the paper's reported values

## Limitations

- Only the three public benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600) are distributed; the record does not cover other instruments or studies
- Optimized per-dataset parameters for the evaluation are reported only in supplementary tables (S1–S3), so downloading the data alone is not sufficient for exact reproduction

## Evidence

- [methods] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [readme] Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034).: "Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)."
- [methods] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [readme] Place the data download from the Zenodo repository as follows.: "Place the data download from the Zenodo repository as follows."
