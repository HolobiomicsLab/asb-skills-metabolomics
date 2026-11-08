---
name: metabolomics-benchmark-dataset-processing
description: Use when benchmarking an untargeted LC-MS/MS processing pipeline against
  published feature lists from public MetaboLights datasets, to quantify what fraction
  of published features (per ionization mode) your workflow recovers and whether quantification
  agrees.
when_to_use_negative:
- Input is already a processed feature table rather than raw/mzML LC-MS/MS data —
  the workflow operates on raw or centroided mzML files
- The dataset has no published reference feature list to benchmark against — the skill's
  core step (matching against XCMS/CAMERA published features) cannot be performed
- Data are Agilent/Bruker vendor files you expect to convert in-workflow — those must
  be converted independently with ProteoWizard first (file conversion rule only handles
  Thermo raw)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: UmetaFlow
  role: Untargeted metabolomics Snakemake workflow performing OpenMS-based feature
    detection, map alignment, grouping, and re-quantification
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: Snakemake
  role: Workflow engine executing the pipeline with conda environments (--use-conda)
    and dry-run configuration validation
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS
  role: Underlying algorithms for feature detection, map alignment, adduct annotation,
    re-quantification and feature linking (OpenMS 3.0 tools)
- name: pyOpenMS
  role: Python bindings to OpenMS used in the Jupyter notebook implementation of the
    same workflow
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow.git
- name: Python
  role: Generates config/dataset.tsv via data_files.py; used for post-hoc matching
    against published feature lists
- name: Jupyter
  role: Notebook implementation (Create_dataset_tsv.ipynb, 2_Preprocessing.ipynb,
    3_Requantification) as alternative to Snakemake execution
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:4577d72b56110776b98abde43088f0068b1fc7bfc8054cfe2e0bbf088c5403c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolomics-benchmark-dataset-processing

## Summary

Reproduce published untargeted metabolomics feature detection/quantification on public MetaboLights datasets (e.g., MTBLS1129/MTBLS1130 colon tumor, Waters UPLC-QTOF) using the UmetaFlow Snakemake/pyOpenMS workflow, then match detected features against published XCMS/CAMERA feature lists to compute detection rates.

## When to use

Use when benchmarking an untargeted LC-MS/MS processing pipeline against published feature lists from public MetaboLights datasets, to quantify what fraction of published features (per ionization mode) your workflow recovers and whether quantification agrees.

## When NOT to use

- Input is already a processed feature table rather than raw/mzML LC-MS/MS data — the workflow operates on raw or centroided mzML files
- The dataset has no published reference feature list to benchmark against — the skill's core step (matching against XCMS/CAMERA published features) cannot be performed
- Data are Agilent/Bruker vendor files you expect to convert in-workflow — those must be converted independently with ProteoWizard first (file conversion rule only handles Thermo raw)

## Inputs

- Raw vendor files (Thermo .raw) or centroided mzML files from public MetaboLights datasets (e.g., MTBLS1129, MTBLS1130 Waters UPLC-QTOF colon tumor data)
- Published XCMS/CAMERA feature lists (m/z, retention time, per-sample intensities) for the benchmark dataset
- config/dataset.tsv listing sample_name and comment for all files to process
- config/blanks.tsv and config/samples.tsv assigning blank/QC vs biological samples

## Outputs

- UmetaFlow feature matrix per ionization mode (m/z, retention time, per-sample intensities) in TSV format under results/
- detection_rate_comparison.csv with fraction of published features detected per mode and intensity correlation across samples

## How to apply

Download the raw/mzML files from MetaboLights and place them under data/raw/ (Thermo raw, converted to centroided mzML in-workflow) or data/mzML/ (Agilent/Bruker or pre-converted files via ProteoWizard). Edit config/config.yaml to set file conversion TRUE/FALSE, define positive/negative ionization modes, and enable the optional re-quantification step; generate config/dataset.tsv with 'python data_files.py' (or the Create_dataset_tsv notebook) and assign blanks/QC to config/blanks.tsv and biological samples to config/samples.tsv. Validate with 'snakemake --use-conda --dry-run', then run 'snakemake --use-conda --cores all', which performs OpenMS 3.0-based feature detection, map alignment, feature grouping/adduct annotation, optional blank/QC removal and minfrac filtering of consensus features with too many missing values, followed by re-quantification of features with missing values across samples. Export the per-mode UmetaFlow feature matrix (m/z, retention time, per-sample intensities), match against the published XCMS/CAMERA feature lists using m/z and RT tolerance matching, and compute the fraction of published features detected per ionization mode plus the correlation of feature intensities across samples. On MTBLS1129 and MTBLS1130 the benchmark target is >= 0.90 detection in positive mode and 1.00 in negative mode with matching intensity patterns.

## Related tools

- **UmetaFlow** (Untargeted metabolomics Snakemake workflow performing OpenMS-based feature detection, map alignment, grouping, and re-quantification) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **Snakemake** (Workflow engine executing the pipeline with conda environments (--use-conda) and dry-run configuration validation) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS** (Underlying algorithms for feature detection, map alignment, adduct annotation, re-quantification and feature linking (OpenMS 3.0 tools))
- **pyOpenMS** (Python bindings to OpenMS used in the Jupyter notebook implementation of the same workflow) — https://github.com/biosustain/pyOpenMS_UmetaFlow.git
- **Python** (Generates config/dataset.tsv via data_files.py; used for post-hoc matching against published feature lists)
- **Jupyter** (Notebook implementation (Create_dataset_tsv.ipynb, 2_Preprocessing.ipynb, 3_Requantification) as alternative to Snakemake execution) — https://github.com/biosustain/pyOpenMS_UmetaFlow.git

## Examples

```
python data_files.py && snakemake --use-conda --dry-run && snakemake --use-conda --cores all
```

## Evaluation signals

- Computed detection rate must be at least 0.90 of published XCMS/CAMERA features in positive mode and 1.00 in negative mode on MTBLS1129/MTBLS1130
- Quantification patterns (intensity distributions across tumor vs normal samples) must agree with the published XCMS/CAMERA results
- Correlation of feature intensities across samples between UmetaFlow and published feature lists is high/similar
- Dry run ('snakemake --use-conda --dry-run') completes without errors, confirming dataset.tsv/blanks.tsv/samples.tsv are consistent with the config.yaml mode and conversion settings
- Exported feature matrices contain m/z, retention time, and per-sample intensity columns for both ionization modes

## Limitations

- File conversion rule supports only Thermo raw files; Agilent/Bruker files must be converted independently using ProteoWizard before being added to data/mzML/
- Re-quantification and blank/QC removal are optional and depend on correct assignment of samples in blanks.tsv/samples.tsv; misassignment changes the feature matrix
- The Snakemake implementation targets Linux and MacOS systems only
- Benchmark performance numbers (Tables S5, S7) are only in supplementary files, not in the main text
- On MacOS Silicon, OpenMS 3.3.0 must be installed manually and code-signed (sudo codesign --force -s - ./bin/*), and tools must be added to PATH

## Evidence

- [results] On MTBLS1129 and MTBLS1130, after parameter optimization, UmetaFlow detected 90% of published features in positive mode and 100% in negative mode: "The optimal parameters for OpenMS (UmetaFlow) for feature detection and quantification of the MTBLS1129 and MTBLS1130 dataset"
- [methods] minfrac filter step: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [methods] blank/QC removal: "This step includes an optional removal of blank/QC samples if defined by the user."
- [readme] execution commands: "Test your configuration by performing a dry-run via

    snakemake --use-conda --dry-run"
- [readme] file conversion scope: "If you have Agilent, Bruker, or other vendor files, skip that step (write "FALSE" for rule file conversion in the config.yaml file - see more under "Configure workflow"), convert them independently"
- [intro] feature detection pipeline: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
