---
name: mzml-feature-extraction-parsing
description: Use when raw LC–MS data in mzML format must be turned into extracted
  feature lists (m/z, RT, area CSV per run) so that feature alignment tools like G-Aligner,
  or its evaluation against manual annotations, can be run.
when_to_use_negative:
- Feature lists are already extracted as CSV with m/z, RT, area columns (e.g. provided
  in the G-Aligner repository's data folders) — skip extraction and go straight to
  alignment
- You need de novo feature detection performed by G-Aligner itself — G-Aligner does
  not detect features from raw mzML; extraction must be done by MZmine2, OpenMS, XCMS
  or MetaPro
- Your data are not LC–MS metabolomics runs (e.g. proteomics or non-MS data) where
  m/z/RT/area feature lists are not the target representation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: G-Aligner
  role: Consumes the extracted CSV feature lists (m/z, RT, area) to build multipartite
    graphs and solve multi-run feature matching; repo also distributes the pre-extracted
    per-software result folders and evaluation scripts
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Feature extraction software producing self-extracted feature lists from mzML
    runs for the MTBLS562/QE_HF/TripleTOF_6600 benchmarks
- name: OpenMS
  role: Feature extraction software producing self-extracted feature lists compared
    against G-Aligner's benchmark
- name: XCMS
  role: Feature extraction software producing self-extracted feature lists compared
    against G-Aligner's benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:3721d1b6421572ec46b22da70fabab7a43dc22c6a78dc4c31b078f2ba878377e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# mzML feature extraction parsing

## Summary

Converts raw mzML LC–MS runs from public metabolomics benchmark datasets (e.g. MTBLS562, QE_HF, TripleTOF_6600) into software-extracted feature lists in CSV format with m/z, RT and area columns, the only input format G-Aligner accepts for downstream multipartite-graph feature alignment.

## When to use

Use when raw LC–MS data in mzML format must be turned into extracted feature lists (m/z, RT, area CSV per run) so that feature alignment tools like G-Aligner, or its evaluation against manual annotations, can be run.

## When NOT to use

- Feature lists are already extracted as CSV with m/z, RT, area columns (e.g. provided in the G-Aligner repository's data folders) — skip extraction and go straight to alignment
- You need de novo feature detection performed by G-Aligner itself — G-Aligner does not detect features from raw mzML; extraction must be done by MZmine2, OpenMS, XCMS or MetaPro
- Your data are not LC–MS metabolomics runs (e.g. proteomics or non-MS data) where m/z/RT/area feature lists are not the target representation

## Inputs

- mzML files of LC–MS metabolomics runs (from Zenodo record 10.5281/zenodo.8313034)
- Raw MS vendor files (Google Drive mirror of the metabolomics datasets)
- Manually annotated feature dataset containing m/z, RT, area information of library analytes (Zenodo)

## Outputs

- Per-run CSV feature lists with m/z, RT and area columns (e.g. stored under data/MTBLS562_results_<software>)
- Feature lists suitable as input to G-Aligner multipartite graph construction and alignment

## How to apply

Download the mzML files of the metabolomics datasets from Zenodo record 10.5281/zenodo.8313034 (raw MS files are alternatively mirrored on Google Drive), then run a feature extraction algorithm (MZmine2, OpenMS, XCMS or MetaPro) over the mzML runs. Serialize the detection/quantification output as one CSV per run containing exactly the m/z, RT and area columns, since G-Aligner's supported input is 'Feature extraction results in csv format, containing m/z, RT and area columns'. Place the extracted lists under the expected directory layout (e.g. data/MTBLS562_results_mzmine2, data/MTBLS562_results_openms, data/MTBLS562_results_xcms, data/MTBLS562_results_metapro) alongside the manually annotated dataset of library analytes (m/z, RT, area) also distributed via Zenodo. Use the optimized per-dataset parameters from supplementary Tables S1–S3 (e.g. Table S3 for MTBLS562) when re-extracting, so the resulting feature lists are comparable to the paper's benchmark. Validation is by schema check (three required columns present) and by downstream evaluation: extracted features are matched against manual annotations to count accurately aligned features and analytes.

## Related tools

- **G-Aligner** (Consumes the extracted CSV feature lists (m/z, RT, area) to build multipartite graphs and solve multi-run feature matching; repo also distributes the pre-extracted per-software result folders and evaluation scripts) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Feature extraction software producing self-extracted feature lists from mzML runs for the MTBLS562/QE_HF/TripleTOF_6600 benchmarks)
- **OpenMS** (Feature extraction software producing self-extracted feature lists compared against G-Aligner's benchmark)
- **XCMS** (Feature extraction software producing self-extracted feature lists compared against G-Aligner's benchmark)

## Examples

```
curl -L -o mtbls562_mzml.zip https://doi.org/10.5281/zenodo.8313034 && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Each extracted CSV contains the three required columns (m/z, RT, area); G-Aligner rejects other input schemas
- Files land in the expected directory layout (data/MTBLS562_results_mzmine2, data/MTBLS562_results_openms, data/MTBLS562_results_xcms, data/MTBLS562_results_metapro, etc.) so the repository benchmark scripts resolve their paths
- Extraction parameters match the per-dataset optimized sets in supplementary Tables S1–S3 (e.g. Table S3 for MTBLS562)
- Downstream check: extracted features can be matched against the manually annotated library-analyte list (m/z, RT, area) to compute feature-level and analyte-level accuracy
- Evaluation scripts (evaluate_mzmine2_galigner.py, evaluate_openms_galigner.py, evaluate_xcms_galigner.py, evaluate_metapro_galigner.py) run without missing-file or schema errors

## Limitations

- G-Aligner accepts only CSV feature lists with m/z, RT and area columns; other formats must be converted first
- Optimized extraction parameters are reported only in supplementary Tables S1–S3, not in the main text
- The manually annotated dataset covers only three public benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600), so accuracy validation of extracted features is limited to those
- Raw data access is split across Zenodo (mzML) and Google Drive (raw MS files), so incomplete downloads are a practical failure mode

## Evidence

- [readme] Feature extraction results in csv format, containing m/z, RT and area columns.: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [readme] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)."
- [intro] features of each run are first detected and quantified from raw data by feature extraction algorithms, and features of multiple related LC–MS runs are then combined into one consensus result map: "features of each run are first detected and quantified from raw data by feature extraction algorithms, and features of multiple related LC–MS runs are then combined into one consensus result map"
- [methods] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [readme] Change the parameters in data/evaluate_metapro_galigner.py: "Change the parameters in data/evaluate_metapro_galigner.py"
