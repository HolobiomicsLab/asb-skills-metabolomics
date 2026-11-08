---
name: benchmark-reproduction-against-ground-truth-features
description: Use when a publicly available LC-MS/MS dataset with a published ground-truth
  feature list (e.g. MetaboLights MTBLS736) must be used to benchmark a feature detection,
  quantification or marker selection pipeline.
when_to_use_negative:
- The input is already a processed feature table and no raw vendor/mzML files are
  available, so feature detection cannot be re-run against ground truth
- No published ground-truth feature list or expected marker set exists for the dataset,
  so detection/quantification/marker performance cannot be scored
- A new instrument dataset without known secondary metabolites is being explored de
  novo rather than benchmarking the workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: 'Workflow under benchmark test: OpenMS 3.0-based feature detection, map alignment,
    adduct annotation, re-quantification and marker selection'
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Snakemake
  role: Executes the reproducible pipeline steps (--use-conda --cores all) and dry-run
    configuration validation
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Provides the algorithms (feature detection, alignment, adduct annotation,
    re-quantification) whose output is scored against ground truth
- name: Python
  role: Utility scripts (data_files.py) and statistical testing for marker selection
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:806a557728fd15add3d97608e6c0cf0f0e4a00a78aaa1b81011be4d96b82935d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# benchmark-reproduction-against-ground-truth-features

## Summary

Run an untargeted metabolomics workflow (UmetaFlow) on a public benchmark dataset such as MTBLS736 or MTBLS733 and score feature detection, quantification, and discriminating marker selection against published ground-truth feature lists.

## When to use

Use when a publicly available LC-MS/MS dataset with a published ground-truth feature list (e.g. MetaboLights MTBLS736) must be used to benchmark a feature detection, quantification or marker selection pipeline.

## When NOT to use

- The input is already a processed feature table and no raw vendor/mzML files are available, so feature detection cannot be re-run against ground truth
- No published ground-truth feature list or expected marker set exists for the dataset, so detection/quantification/marker performance cannot be scored
- A new instrument dataset without known secondary metabolites is being explored de novo rather than benchmarking the workflow

## Inputs

- Public benchmark raw LC-MS/MS files from MetaboLights (e.g. MTBLS733, MTBLS736, AB SCIEX TripleTOF 6600 data)
- Ground-truth feature list for the dataset (e.g. 970 known features on MTBLS736)
- config/dataset.tsv, config/blanks.tsv, config/samples.tsv sample tables
- config/config.yaml with ionisation mode and pipeline toggles

## Outputs

- Annotated feature matrix after OpenMS 3.0 pre-processing and re-quantification
- Discriminating marker list from statistical testing between sample groups
- Benchmark scores: percentage of ground-truth features detected, quantification accuracy rate, true/false marker counts
- Optional GNPS FBMN/IIMN GraphML and SIRIUS/CSI:FingerID annotations

## How to apply

Download the benchmark raw files from MetaboLights into data/raw/, generate config/dataset.tsv with 'python data_files.py', and split blanks and true samples into config/blanks.tsv and config/samples.tsv. Set dataset-specific parameters in config/config.yaml (ionisation mode, pipeline steps enabled), validate with 'snakemake --use-conda --dry-run', then execute 'snakemake --use-conda --cores all' inside the umetaflow-snakemake conda environment so files are converted to centroid mzML and OpenMS 3.0 tools perform feature detection, map alignment, adduct annotation, blank/QC removal, minfrac filtering and re-quantification. Apply statistical testing on the annotated feature matrix to select discriminating markers between sample groups (e.g. SA vs SB on MTBLS736). Compare outputs to ground truth: for MTBLS736 expect 874 of 970 annotated features (90.1% true feature ID rate), ~81.7% accurately quantified features, and 59 true versus 1 false discriminating markers.

## Related tools

- **UmetaFlow** (Workflow under benchmark test: OpenMS 3.0-based feature detection, map alignment, adduct annotation, re-quantification and marker selection) — https://github.com/biosustain/snakemake_UmetaFlow
- **Snakemake** (Executes the reproducible pipeline steps (--use-conda --cores all) and dry-run configuration validation) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Provides the algorithms (feature detection, alignment, adduct annotation, re-quantification) whose output is scored against ground truth)
- **Python** (Utility scripts (data_files.py) and statistical testing for marker selection) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Annotated feature count reaches >90% of ground truth (e.g. 874 of 970 features on MTBLS736)
- ~81.7% of features are accurately quantified against the published quantification benchmark
- Marker selection recovers 59 of 68 true discriminating markers with at most 1 false positive on MTBLS736
- snakemake --use-conda --dry-run completes without errors, confirming config/dataset.tsv, blanks.tsv and samples.tsv are consistent
- All raw files converted to centroid mzML before pre-processing; re-quantification applied to features with missing values

## Limitations

- Benchmark performance numbers for MTBLS733 (Tables S5, S7) are only in supplementary files, so scoring thresholds must be read from the SI
- No explicit software version numbers are stated for UmetaFlow, OpenMS, Snakemake or SIRIUS/CSI:FingerID in the article text, complicating exact replication
- minfrac filtering may remove consensus features with too many missing values and thus lower detection counts if set aggressively
- File conversion step only handles Thermo raw files natively; Agilent/Bruker files must be converted independently with proteowizard and placed under data/mzML/

## Evidence

- [results] 90.1% true feature ID rate on MTBLS736: [evidence span withheld — anchor status: not_found]
- [discussion] benchmark datasets named: "validated UmetaFlow with in-house data, and benchmarked it using the publicly available datasets MTBLS733 and MTBLS736"
- [intro] OpenMS 3.0 toolchain used: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [readme] README: blank/QC and minfrac steps: "This step includes an optional removal of blank/QC samples if defined by the user. Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] README: execution command: "snakemake --use-conda --cores all"
