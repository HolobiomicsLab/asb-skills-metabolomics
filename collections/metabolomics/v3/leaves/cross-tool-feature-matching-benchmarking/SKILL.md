---
name: cross-tool-feature-matching-benchmarking
description: Use when reproducing published untargeted metabolomics feature detection
  with a different tool and quantifying what fraction of the published (XCMS/CAMERA)
  features is recovered, separately per ionization mode, from public MetaboLights
  raw data.
when_to_use_negative:
- Input is already a feature/consensus table — this skill starts from vendor raw or
  centroided mzML files and requires running feature detection first
- No published or ground-truth feature list exists for the dataset, so no cross-tool
  recall fraction can be computed
- The goal is molecular annotation (SIRIUS/CSI:FingerID/GNPS FBMN) rather than benchmarking
  feature-detection and quantification coverage
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: Snakemake/pyOpenMS workflow whose OpenMS 3.0 feature detection, map alignment,
    feature grouping, and re-quantification outputs are benchmarked against the published
    XCMS/CAMERA features
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Provides the feature detection, map alignment, adduct annotation, re-quantification
    and feature linking algorithms invoked by UmetaFlow
- name: pyOpenMS
  role: Python bindings used in the Jupyter-notebook implementation of the same UmetaFlow
    pipeline for Linux, Windows and MacOS
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Python
  role: Used to generate config/dataset.tsv (data_files.py) and to perform m/z/RT
    tolerance matching and detection-fraction/correlation computation against the
    published feature lists
- name: XCMS
  role: The published reference processing tool whose feature lists (with CAMERA)
    on MTBLS1129/MTBLS1130 define the ground truth for the detection-rate comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:ee8016ac771e2260e4e0c7066039e09d97c6051c25d640ae2821244047900edf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - Jupyter
  - OpenMS
  - Python
  - UmetaFlow
  - XCMS
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# cross-tool-feature-matching-benchmarking

## Summary

Benchmark an alternative untargeted metabolomics feature-detection workflow (e.g., OpenMS-based UmetaFlow) against a published XCMS/CAMERA analysis by matching detected features to published feature lists via m/z and retention-time tolerance, computing per-mode detection fractions and intensity-pattern agreement.

## When to use

Use when reproducing published untargeted metabolomics feature detection with a different tool and quantifying what fraction of the published (XCMS/CAMERA) features is recovered, separately per ionization mode, from public MetaboLights raw data.

## When NOT to use

- Input is already a feature/consensus table — this skill starts from vendor raw or centroided mzML files and requires running feature detection first
- No published or ground-truth feature list exists for the dataset, so no cross-tool recall fraction can be computed
- The goal is molecular annotation (SIRIUS/CSI:FingerID/GNPS FBMN) rather than benchmarking feature-detection and quantification coverage

## Inputs

- Centroided mzML files for MTBLS1129 and MTBLS1130 (MetaboLights, Waters UPLC-QTOF), per ionization mode (positive and negative)
- Published XCMS/CAMERA feature lists (m/z, retention time, per-sample intensities) as the reference/ground truth
- config/config.yaml, config/dataset.tsv, config/blanks.tsv, config/samples.tsv defining conversion, ionization modes, and sample/blank assignments

## Outputs

- UmetaFlow feature matrix per ionization mode (m/z, retention time, per-sample intensities)
- detection_rate_comparison.csv with detection fractions for positive and negative mode
- Feature intensity correlations and tumor-vs-normal intensity distribution comparison against the published XCMS/CAMERA results

## How to apply

Obtain the public dataset raw files (e.g., MTBLS1129 and MTBLS1130, Waters UPLC-QTOF colon tumor data) and place Thermo raw files in data/raw/ or already-converted centroided mzML files in data/mzML/ (convert Agilent/Bruker independently with ProteoWizard). Configure config/config.yaml with file conversion TRUE, both ionization modes (positive and negative), and the re-quantification step enabled, generate config/dataset.tsv via 'python data_files.py' or the Create_dataset_tsv notebook, and assign blanks/QC to blanks.tsv and biological samples to samples.tsv. Validate with 'snakemake --use-conda --dry-run', then run 'snakemake --use-conda --cores all', which performs OpenMS 3.0 feature detection, map alignment, feature grouping with optional blank/QC removal and minfrac filtering of consensus features with too many missing values, followed by re-quantification of features missing across samples. Export the UmetaFlow feature matrix (m/z, RT, per-sample intensities) per ionization mode and match features against the published XCMS/CAMERA feature lists using m/z and RT tolerance matching; compute the fraction of published features detected per mode and the correlation of feature intensities across samples. Report results in detection_rate_comparison.csv, applying the acceptance thresholds of ≥0.90 detection in positive mode and 1.00 in negative mode, and verify that intensity distributions across tumor vs. normal samples agree with the published quantification patterns.

## Related tools

- **UmetaFlow** (Snakemake/pyOpenMS workflow whose OpenMS 3.0 feature detection, map alignment, feature grouping, and re-quantification outputs are benchmarked against the published XCMS/CAMERA features) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Provides the feature detection, map alignment, adduct annotation, re-quantification and feature linking algorithms invoked by UmetaFlow)
- **pyOpenMS** (Python bindings used in the Jupyter-notebook implementation of the same UmetaFlow pipeline for Linux, Windows and MacOS) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Python** (Used to generate config/dataset.tsv (data_files.py) and to perform m/z/RT tolerance matching and detection-fraction/correlation computation against the published feature lists)
- **XCMS** (The published reference processing tool whose feature lists (with CAMERA) on MTBLS1129/MTBLS1130 define the ground truth for the detection-rate comparison)

## Examples

```
snakemake --use-conda --dry-run && snakemake --use-conda --cores all
```

## Evaluation signals

- Computed detection rate is ≥0.90 in positive mode on MTBLS1129/MTBLS1130 (recorded in detection_rate_comparison.csv)
- Computed detection rate is 1.00 in negative mode on MTBLS1129/MTBLS1130
- Correlation of feature intensities across samples matches the published quantification patterns
- Intensity distributions across tumor vs. normal samples agree with the published XCMS/CAMERA results
- Snakemake dry run validates configuration (dataset.tsv, blanks.tsv, samples.tsv) before the full 'snakemake --use-conda --cores all' execution

## Limitations

- Detection ≥90% was only achieved after parameter optimization of the OpenMS/UmetaFlow settings for the MTBLS1129 and MTBLS1130 datasets, so defaults may underperform
- Detailed benchmark performance numbers (Tables S5, S7) are only available in supplementary files, not in the main text
- No software version numbers are stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake, hindering exact reproduction; the paper's headline benchmark (>90% of ground-truth features) is from MTBLS733/MTBLS736, while the 90%/100% figures are the colon-tumor datasets

## Evidence

- [abstract] Benchmark detection threshold on MTBLS733/736: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [discussion] MTBLS1129 cross-tool comparison in paper: "detected and quantified with (a) XCMS and (b) UmetaFlow (dataset MTBLS1129)"
- [discussion] Parameter optimization requirement: "The optimal parameters for OpenMS (UmetaFlow) for feature detection and quantification of the MTBLS1129 and MTBLS1130 dataset"
- [methods] Optional filter steps affecting feature counts: "This step includes an optional removal of blank/QC samples if defined by the user. Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] mzML input handling: "If you have Agilent, Bruker, or other vendor files, skip that step (write "FALSE" for rule file conversion in the config.yaml file"
