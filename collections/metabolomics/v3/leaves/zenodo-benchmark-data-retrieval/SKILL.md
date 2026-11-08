---
name: zenodo-benchmark-data-retrieval
description: 'Use when reproducing G-Aligner benchmark evaluations or any task requiring
  the QE_HF/MTBLS562/TripleTOF_6600 datasets: the deposit DOI 10.5281/zenodo.8313034
  supplies mzML raw files, manual annotations and evaluation scripts that must be
  laid out per the README directory tree.'
when_to_use_negative:
- The task needs raw vendor-format MS files rather than mzML — those live on a separate
  Google Drive link, not the Zenodo deposit
- The analysis targets datasets outside the three benchmarks (QE_HF, MTBLS562, TripleTOF_6600);
  the deposit only covers those, and no optimized parameters beyond supplementary
  Tables S1–S3 are provided
- Only feature alignment output is needed and the data layout already exists — re-downloading
  the deposit adds nothing
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: G-Aligner
  role: The open-source package whose repository README defines the required data
    layout and whose benchmark scripts consume the retrieved Zenodo deposit
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison software whose self-extracted feature results (*_results_mzmine2
    folders from the deposit) sit alongside the retrieved mzML data for later evaluation
- name: OpenMS
  role: Comparison software whose *_results_openms folders from the deposit are staged
    by this retrieval step
- name: XCMS
  role: Comparison software whose *_results_xcms folders from the deposit are staged
    by this retrieval step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:18991d81999a789ae3d7310c7b0bec32442149f39a7b0d35503c07fce111c702
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# zenodo-benchmark-data-retrieval

## Summary

Retrieves the G-Aligner benchmark deposit (Zenodo 10.5281/zenodo.8313034) containing mzML files, manually annotated feature lists (m/z, RT, area), per-software results and evaluation scripts, and arranges it under ROOT_PATH/data (e.g. QE_HF, QE_HF_results_metapro) to enable faithful reproduction of the paper's alignment-accuracy evaluations.

## When to use

Use when reproducing G-Aligner benchmark evaluations or any task requiring the QE_HF/MTBLS562/TripleTOF_6600 datasets: the deposit DOI 10.5281/zenodo.8313034 supplies mzML raw files, manual annotations and evaluation scripts that must be laid out per the README directory tree.

## When NOT to use

- The task needs raw vendor-format MS files rather than mzML — those live on a separate Google Drive link, not the Zenodo deposit
- The analysis targets datasets outside the three benchmarks (QE_HF, MTBLS562, TripleTOF_6600); the deposit only covers those, and no optimized parameters beyond supplementary Tables S1–S3 are provided
- Only feature alignment output is needed and the data layout already exists — re-downloading the deposit adds nothing

## Inputs

- Zenodo deposit DOI 10.5281/zenodo.8313034 (benchmark datasets, manual annotation results, evaluation methods and results)
- mzML files of the metabolomics datasets (QE_HF, MTBLS562, TripleTOF_6600)
- Manually annotated feature lists (m/z, RT, area of library analytes)
- Evaluation and comparison scripts shipped in the deposit (e.g. evaluate_metapro_galigner.py, metapro_result_comparison.py, software_result_comparison.py)
- Raw MS vendor files via the project's Google Drive link (optional, if mzML sufficiency is not assumed)

## Outputs

- Populated ROOT_PATH/data directory tree: QE_HF/, MTBLS562/, TripleTOF_6600/ plus their *_results_metapro, *_results_mzmine2, *_results_openms, *_results_xcms folders
- Evaluation scripts and feature-comparison scripts in place under ROOT_PATH/data, ready to execute from ROOT_PATH

## How to apply

Download the deposit identified by DOI 10.5281/zenodo.8313034, which bundles the mzML files, targeted (manually annotated) feature lists containing m/z, RT and area information of library analytes, evaluation scripts (evaluate_metapro_galigner.py, evaluate_metapro_mzmine2.java, evaluate_metapro_openms.py, evaluate_metapro_xcms.Rmd, etc.) and result comparison scripts. Arrange the download exactly as the README tree specifies: each dataset (QE_HF, MTBLS562, TripleTOF_6600) gets its own folder plus *_results_metapro, *_results_mzmine2, *_results_openms and *_results_xcms siblings under data/. Raw MS files (not mzML) are provided via a separate Google Drive link if needed. Correct placement is a prerequisite for running metapro_result_comparison.py and software_result_comparison.py from ROOT_PATH, and for evaluate_metapro_galigner.py to write alignment results into the 'experiment' folder.

## Related tools

- **G-Aligner** (The open-source package whose repository README defines the required data layout and whose benchmark scripts consume the retrieved Zenodo deposit) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison software whose self-extracted feature results (*_results_mzmine2 folders from the deposit) sit alongside the retrieved mzML data for later evaluation)
- **OpenMS** (Comparison software whose *_results_openms folders from the deposit are staged by this retrieval step)
- **XCMS** (Comparison software whose *_results_xcms folders from the deposit are staged by this retrieval step)

## Examples

```
mkdir -p ROOT_PATH/data/QE_HF ROOT_PATH/data/QE_HF_results_metapro  # extract Zenodo 10.5281/zenodo.8313034 download into this README-specified layout
```

## Evaluation signals

- All dataset directories exist as siblings under ROOT_PATH/data: QE_HF, QE_HF_results_metapro, QE_HF_results_mzmine2, QE_HF_results_openms, QE_HF_results_xcms (and MTBLS562/TripleTOF_6600 equivalents)
- Downloaded mzML files parse as valid mzML and correspond to the three public benchmark datasets
- Manually annotated feature lists contain the expected m/z, RT and area columns of library analytes
- Evaluation scripts (evaluate_metapro_galigner.py, metapro_result_comparison.py, software_result_comparison.py) are present under ROOT_PATH/data and can be executed from ROOT_PATH
- Downstream G-Aligner runs succeed and write feature alignment results into the 'experiment' folder, confirming the layout matches the scripts' expectations

## Limitations

- Raw MS files are distributed via a Google Drive folder while mzML and annotations come from Zenodo, so a single-deposit download is not guaranteed to be complete
- Optimized evaluation parameters are reported only in supplementary Tables S1–S3, not in the main text, and must be fetched separately when configuring evaluate_metapro_galigner.py
- The deposit supports only the three benchmark datasets; extension to other datasets requires manual annotation work not covered by the retrieval step

## Evidence

- [abstract] Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034: "Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034"
- [methods] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [readme] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes and can be used in evaluations of feature detection,"
- [readme] Place the data download from the Zenodo repository as follows.: "Our demos can help you reproduce the evaluation results.

Place the data download from the Zenodo repository as follows."
- [methods] Feature alignment results are saved in ```experiment``` folder.: "Feature alignment results are saved in ```experiment``` folder."
