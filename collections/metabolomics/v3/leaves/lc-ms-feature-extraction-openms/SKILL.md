---
name: lc-ms-feature-extraction-openms
description: Use when raw mzML LC-MS runs (e.g. MTBLS562, QE_HF, TripleTOF_6600 benchmark
  datasets) must be converted into per-run feature lists (m/z, RT, area) for downstream
  alignment or for reproducing an OpenMS-based baseline.
when_to_use_negative:
- Pre-extracted OpenMS feature tables already exist (e.g. the deposited MTBLS562_results_openms,
  QE_HF_results_openms, TripleTOF_6600_results_openms folders) — use those directly
  instead of re-running feature finding.
- The goal is only to evaluate alignment of manually annotated analyte features —
  the manual annotation dataset with m/z, RT, area of library analytes is deposited
  and does not require re-extraction.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: OpenMS
  role: Feature-finding modules used to detect and quantify untargeted features from
    raw mzML runs; its native feature-grouping module also provides the baseline alignment
    that extraction feeds.
- name: G-Aligner
  role: Downstream consumer of the extracted CSV feature lists (m/z, RT, area) for
    re-alignment comparison against the OpenMS baseline.
  repo: https://github.com/CSi-Studio/G-Aligner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:63477f8e78156cf294f7288148621e5703f26ff7b45540237292cc3ffcd795a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - OpenMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-feature-extraction-openms

## Summary

Extracts untargeted features from raw LC-MS runs (mzML) using OpenMS's feature-finding modules, producing per-run feature lists with m/z, RT and intensity/area columns. In the G-Aligner benchmark this supplies the OpenMS self-extracted features used to establish the native-alignment baseline and as alignment input.

## When to use

Use when raw mzML LC-MS runs (e.g. MTBLS562, QE_HF, TripleTOF_6600 benchmark datasets) must be converted into per-run feature lists (m/z, RT, area) for downstream alignment or for reproducing an OpenMS-based baseline.

## When NOT to use

- Pre-extracted OpenMS feature tables already exist (e.g. the deposited MTBLS562_results_openms, QE_HF_results_openms, TripleTOF_6600_results_openms folders) — use those directly instead of re-running feature finding.
- The goal is only to evaluate alignment of manually annotated analyte features — the manual annotation dataset with m/z, RT, area of library analytes is deposited and does not require re-extraction.

## Inputs

- mzML files of the MTBLS562, QE_HF and TripleTOF_6600 metabolomics datasets (Zenodo record 10.5281/zenodo.8313034)
- OpenMS feature-finding parameter settings per dataset (supplementary Tables S1–S3)

## Outputs

- Per-run CSV feature lists containing m/z, RT and intensity/area columns for each LC-MS run
- OpenMS baseline alignment results (input to F_ACC / A_ACC evaluation)

## How to apply

Download the mzML files of the MTBLS562, QE_HF and TripleTOF_6600 datasets from Zenodo record 10.5281/zenodo.8313034. Run the OpenMS feature-finding modules on each run, following the paper's OpenMS workflow, so that features of each run are detected and quantified from the raw data. Export the results as per-run CSV feature lists containing m/z, RT and area columns — the format G-Aligner and the evaluation scripts expect. These extracted lists serve two roles: (1) they are aligned with OpenMS's native feature-grouping module to establish the OpenMS baseline (F_ACC/A_ACC), and (2) they are re-aligned by G-Aligner using the m/z and RT tolerances from the supplementary parameter tables (S1–S3) for each dataset. Note that extraction parameters are dataset-specific and reported only in the supplementary tables.

## Related tools

- **OpenMS** (Feature-finding modules used to detect and quantify untargeted features from raw mzML runs; its native feature-grouping module also provides the baseline alignment that extraction feeds.)
- **G-Aligner** (Downstream consumer of the extracted CSV feature lists (m/z, RT, area) for re-alignment comparison against the OpenMS baseline.) — https://github.com/CSi-Studio/G-Aligner

## Evaluation signals

- Each output feature list is a CSV containing m/z, RT and area columns, matching the format G-Aligner accepts.
- One feature list is produced per mzML input run for each of the three datasets (MTBLS562, QE_HF, TripleTOF_6600).
- Aligning the extracted lists with OpenMS's native feature-grouping module yields a baseline whose F_ACC/A_ACC gains under G-Aligner (~2.4%, 1.5%, 1.7% F_ACC; ~13.5%, 10.6%, 18.4% A_ACC) match the reported values within tolerance (expert_review).
- The evaluation table can be regenerated with the deposited scripts (e.g. evaluate_openms_galigner.py, software_result_comparison.py) against the manually annotated feature dataset.
- Extraction parameters used match the dataset-specific optimized parameters in supplementary Tables S1–S3.

## Limitations

- Optimized per-dataset parameters are reported only in supplementary tables (S1–S3), not in the main text, so the exact OpenMS extraction settings must be recovered from the SI.
- Extraction results are workflow- and software-specific: the baseline accuracy depends on OpenMS's own feature finder, so features extracted with another tool are not comparable in this benchmark.
- Raw MS vendor files are hosted on Google Drive while only mzML conversions are on Zenodo; results rely on the deposited mzML conversions being used.

## Evidence

- [intro] features of each run are first detected and quantified from raw data by feature extraction algorithms: "features of each run are first detected and quantified from raw data by feature extraction algorithms"
- [readme] Feature extraction rsults in csv format, containing m/z, RT and area columns.: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [methods] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [readme] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
