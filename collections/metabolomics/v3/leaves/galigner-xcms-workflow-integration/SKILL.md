---
name: galigner-xcms-workflow-integration
description: Use when XCMS feature extraction results (csv feature lists with m/z,
  RT, area) must be aligned across LC-MS runs more accurately than XCMS's native method,
  and the gain must be benchmarked against manually annotated library analytes.
when_to_use_negative:
- Input is raw mzML data without prior feature extraction — G-Aligner requires feature
  extraction results in csv format with m/z, RT and area columns, not raw spectra
- The goal is alignment of MZmine2, OpenMS or MetaPro extracted features — those require
  the corresponding evaluate_mzmine2_galigner.py / evaluate_openms_galigner.py / evaluate_metapro_galigner.py
  scripts, not the XCMS integration path
- Runtime is the binding constraint — the multidimensional assignment problem is NP-Hard
  and G-Aligner's main limitation is lack of computing speed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: 'Alignment engine: builds a multipartite graph from XCMS self-extracted features,
    performs RANSAC/OBI-Warp RT coarse registration, and solves the unbalanced multidimensional
    assignment problem with three combinatorial optimization solvers'
  repo: https://github.com/CSi-Studio/G-Aligner
- name: XCMS
  role: Upstream feature extraction software whose self-extracted untargeted feature
    lists are aligned by G-Aligner; its native alignment results serve as the comparison
    baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:97db844fd1e4094592afbcdca818cb1ad1e2afaa61b02b4b6a8ab26ce22209ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# galigner-xcms-workflow-integration

## Summary

Integrates G-Aligner, a multipartite-graph feature alignment method, into an XCMS analysis workflow to align XCMS self-extracted untargeted features, and quantifies F_ACC/A_ACC improvement over XCMS's native alignment on the TripleTOF 6600, QE HF and MTBLS562 benchmark datasets.

## When to use

Use when XCMS feature extraction results (csv feature lists with m/z, RT, area) must be aligned across LC-MS runs more accurately than XCMS's native method, and the gain must be benchmarked against manually annotated library analytes.

## When NOT to use

- Input is raw mzML data without prior feature extraction — G-Aligner requires feature extraction results in csv format with m/z, RT and area columns, not raw spectra
- The goal is alignment of MZmine2, OpenMS or MetaPro extracted features — those require the corresponding evaluate_mzmine2_galigner.py / evaluate_openms_galigner.py / evaluate_metapro_galigner.py scripts, not the XCMS integration path
- Runtime is the binding constraint — the multidimensional assignment problem is NP-Hard and G-Aligner's main limitation is lack of computing speed

## Inputs

- mzML files of the three benchmark metabolomics datasets (TripleTOF 6600, QE HF, MTBLS562) from Zenodo doi:10.5281/zenodo.8313034
- XCMS feature extraction results (csv feature lists containing m/z, RT and area columns) per dataset
- Manually annotated feature dataset with m/z, RT and area information of library analytes
- Optimized G-Aligner parameters per dataset (supplementary Tables S1–S3)

## Outputs

- G-Aligner-aligned consensus feature lists for the XCMS self-extracted features (saved in the experiment folder)
- F_ACC and A_ACC of G-Aligner-on-XCMS versus native XCMS per dataset
- Per-dataset improvement in feature accuracy and analyte accuracy over XCMS's native alignment

## How to apply

Download the mzML files, XCMS feature extraction results and manual annotation results from Zenodo 10.5281/zenodo.8313034 and organize them under data/TripleTOF_6600_results_xcms, data/QE_HF_results_xcms and data/MTBLS562_results_xcms as specified in the README. Run G-Aligner on the XCMS self-extracted feature lists — G-Aligner builds a multipartite graph of features and potential correspondences, performs retention-time coarse registration with RANSAC and OBI-Warp, then solves the multi-run matching as an unbalanced multidimensional assignment problem with three combinatorial optimization solvers, using the optimized per-dataset parameters (Tables S1–S3). Run data/evaluate_xcms_galigner.py to compute feature accuracy (F_ACC) and analyte accuracy (A_ACC) of the aligned features against the manually annotated library analytes, then run data/software_result_comparison.py to compare against native XCMS alignment. Record the per-dataset F_ACC and A_ACC gains and check them against the reported improvements (+9.8%, +2.6%, +9.4% F_ACC; +24.5%, +11.4%, +24.7% A_ACC for TripleTOF 6600, QE HF and MTBLS562).

## Related tools

- **G-Aligner** (Alignment engine: builds a multipartite graph from XCMS self-extracted features, performs RANSAC/OBI-Warp RT coarse registration, and solves the unbalanced multidimensional assignment problem with three combinatorial optimization solvers) — https://github.com/CSi-Studio/G-Aligner
- **XCMS** (Upstream feature extraction software whose self-extracted untargeted feature lists are aligned by G-Aligner; its native alignment results serve as the comparison baseline)

## Examples

```
python data/evaluate_xcms_galigner.py; python data/software_result_comparison.py
```

## Evaluation signals

- Computed F_ACC improvement over native XCMS matches the reported values per dataset: +9.8% (TripleTOF 6600), +2.6% (QE HF), +9.4% (MTBLS562)
- Computed A_ACC improvement over native XCMS matches the reported values per dataset: +24.5% (TripleTOF 6600), +11.4% (QE HF), +24.7% (MTBLS562)
- G-Aligner-on-XCMS F_ACC and A_ACC must exceed native XCMS alignment on all three benchmark datasets (flag any dataset whose gains deviate from Tables 4, 5 and 6)
- software_result_comparison.py runs without error across all three *_results_xcms folders and produces per-dataset comparisons
- Alignment results are written to the experiment folder of the G-Aligner root directory

## Limitations

- The main limitation of G-Aligner is lack of computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computing time; the two provided acceleration methods still leave a gap to painless application
- GPU acceleration is not yet implemented, so parallel calculations cannot currently be sped up that way
- Optimized per-dataset parameters are documented only in supplementary Tables S1–S3, not in the main text

## Evidence

- [abstract] Integrating G-Aligner into comparison software workflows helped all comparison software obtain more accurate results on their self-extracted features: "helped all comparison software obtain more accurate results on their self-extracted features by integrating G-Aligner to their analysis workflow"
- [intro] RT alignment stage of G-Aligner: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [methods] Benchmark data source: "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [readme] Accepted input format for G-Aligner: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [discussion] Computational cost limitation: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
- [readme] Evaluation script used in this skill: "evaluate_xcms_galigner.py"
