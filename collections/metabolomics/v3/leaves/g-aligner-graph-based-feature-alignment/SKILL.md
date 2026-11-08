---
name: g-aligner-graph-based-feature-alignment
description: Use when aligning untargeted LC-MS feature lists (m/z, RT, area from
  each run) across multiple runs where nonlinear RT drifts and naive nearest-neighbor
  matching degrade accuracy, and when improving OpenMS, MZmine2, or XCMS alignment
  results.
when_to_use_negative:
- Input is raw LC-MS data that has not yet been feature-extracted — G-Aligner requires
  per-run csv feature lists with m/z, RT and area columns, not spectra
- 'Speed-critical production pipelines: the multidimensional assignment problem is
  NP-Hard and the main limitation is lack of computing speed (GPU acceleration not
  yet implemented)'
- Datasets outside the three benchmarks without optimized parameters from supplementary
  tables S1–S3 — parameters are dataset-specific
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: G-Aligner
  role: 'Primary alignment method: multipartite-graph feature matching via unbalanced
    multidimensional assignment after RANSAC/OBI-Warp RT coarse registration'
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OpenMS
  role: 'Baseline and comparison workflow: native feature-grouping alignment provides
    the baseline F_ACC/A_ACC that G-Aligner integration improves upon; also a source
    of self-extracted features'
- name: MZmine2
  role: 'Comparison method: RANSAC RT alignment plus reference-run pairwise nearest
    matching baseline'
- name: XCMS
  role: 'Comparison method: clustering-based multi-run feature matching baseline'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:fab9d83b85762a5dbadd92b58d6667ac7b0b94944391205c195ea543f53ef5bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# g-aligner-graph-based-feature-alignment

## Summary

Apply G-Aligner, a graph-based feature alignment method for untargeted LC-MS metabolomics that models features as nodes and potential correspondences as edges of a multipartite graph, and solves multi-run feature matching as an unbalanced multidimensional assignment problem using three combinatorial optimization algorithms.

## When to use

Use when aligning untargeted LC-MS feature lists (m/z, RT, area from each run) across multiple runs where nonlinear RT drifts and naive nearest-neighbor matching degrade accuracy, and when improving OpenMS, MZmine2, or XCMS alignment results.

## When NOT to use

- Input is raw LC-MS data that has not yet been feature-extracted — G-Aligner requires per-run csv feature lists with m/z, RT and area columns, not spectra
- Speed-critical production pipelines: the multidimensional assignment problem is NP-Hard and the main limitation is lack of computing speed (GPU acceleration not yet implemented)
- Datasets outside the three benchmarks without optimized parameters from supplementary tables S1–S3 — parameters are dataset-specific

## Inputs

- mzML files of the MTBLS562, QE_HF and TripleTOF_6600 metabolomics benchmark datasets (from Zenodo 10.5281/zenodo.8313034)
- Per-run feature extraction results in csv format containing m/z, RT and area columns
- Manually annotated feature dataset with m/z, RT, area information of library analytes (for evaluation)

## Outputs

- Consensus feature alignment results saved in the experiment folder
- F_ACC and A_ACC accuracy metrics computed against manual annotations
- Comparison metrics tables vs OpenMS, MZmine2 and XCMS baselines (e.g. via software_result_comparison.py)

## How to apply

Download mzML files of the MTBLS562, QE_HF and TripleTOF_6600 benchmark datasets from Zenodo record 10.5281/zenodo.8313034 and extract untargeted features per run (csv with m/z, RT and area columns). Set up the environment by running setup.py install in third_party/ and pip install -r requirements.txt at the G-Aligner root. Run G-Aligner with the dataset-specific optimized parameters from supplementary tables S1–S3: it first performs retention time coarse registration using RANSAC and OBI-Warp, then builds a multipartite graph of features and potential correspondences and solves the unbalanced multidimensional assignment problem with one of three combinatorial optimization algorithms to produce consensus features. Results are saved in the experiment folder; evaluate against manually annotated feature datasets using the deposited scripts (e.g. evaluate_openms_galigner.py) computing F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes), then compare to OpenMS/MZmine2/XCMS baselines.

## Related tools

- **G-Aligner** (Primary alignment method: multipartite-graph feature matching via unbalanced multidimensional assignment after RANSAC/OBI-Warp RT coarse registration) — https://github.com/CSi-Studio/G-Aligner
- **OpenMS** (Baseline and comparison workflow: native feature-grouping alignment provides the baseline F_ACC/A_ACC that G-Aligner integration improves upon; also a source of self-extracted features)
- **MZmine2** (Comparison method: RANSAC RT alignment plus reference-run pairwise nearest matching baseline)
- **XCMS** (Comparison method: clustering-based multi-run feature matching baseline)

## Examples

```
pip install -r requirements.txt; python data/software_result_comparison.py; python data/evaluate_openms_galigner.py
```

## Evaluation signals

- Computed F_ACC gains over OpenMS on self-extracted features must match reported ~2.4%, 1.5% and 1.7% on MTBLS562, QE_HF and TripleTOF_6600 (within reasonable tolerance)
- Computed A_ACC gains must match reported ~13.5%, 10.6% and 18.4% on the same three datasets
- Alignment output files appear in the experiment folder after running the demo/evaluate scripts
- G-Aligner's F_ACC/A_ACC should exceed the OpenMS, MZmine2 and XCMS baselines on all three benchmark datasets (paper reports up to 9.8% and 26.6% increases in accurately aligned features and analytes)
- Feature alignment results must contain consensus features with consistent m/z, RT and area correspondence across runs

## Limitations

- Main limitation is lack of computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computing time; two acceleration methods exist but 'there was still a gap between painless application'
- GPU acceleration for parallel calculations is not yet implemented
- Optimized parameters for each dataset are only reported in supplementary tables S1–S3 (e.g. Table S1 for TripleTOF 6600, Table S3 for MTBLS562), not in the main text

## Evidence

- [readme] G-Aligner treats features and potential correspondences as nodes and edges of a multipartite graph, converts the feature matching problem as a multidimensional assignment problem (MAP): "G-Aligner treats features and potential correspondences as nodes and edges of a multipartite graph, converts the feature matching problem as a multidimensional assignment problem (MAP)"
- [intro] In the retention time alignment stage, G-Aligner uses the existing RANSAC and OBI-Warp algorithms to perform stable coarse registration between runs.: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [readme] Feature extraction results in csv format, containing m/z, RT and area columns.: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [discussion] The main limitation of G-Aligner is the lack of computing speed.: "The main limitation of G-Aligner is the lack of computing speed."
