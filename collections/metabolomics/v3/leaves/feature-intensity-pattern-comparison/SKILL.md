---
name: feature-intensity-pattern-comparison
description: Use when re-processed LC-MS/MS features (m/z, RT, per-sample intensities)
  must be matched against a published XCMS/CAMERA feature list to compute detection
  rates and intensity-pattern agreement, e.g., on MetaboLights datasets like MTBLS1129/MTBLS1130.
when_to_use_negative:
- Input is already a feature table with no raw/mzML data available, since re-quantification
  and blank/QC handling require re-running the pre-processing on raw files
- There is no published or ground-truth feature list to match against, so no detection-rate
  or intensity-pattern comparison is possible
- The dataset is GC-MS or MS-free statistics work, outside UmetaFlow's LC-MS/MS scope
  (Thermo/Agilent/Bruker raw or mzML inputs)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: Re-detects, aligns, groups and re-quantifies features so their intensities
    can be compared with the published XCMS/CAMERA feature lists
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS
  role: Provides the feature detection, map alignment, re-quantification and feature
    linking algorithms executed by UmetaFlow
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: Snakemake
  role: Executes the reproducible workflow (dry-run validation, then full run with
    --use-conda --cores all) that produces the feature matrices to compare
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: pyOpenMS
  role: Python bindings used in the Jupyter-notebook implementation of the same workflow
    for exporting and comparing feature tables
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:973fd82e1d51cdf88ca459e5e16a46f02484adff94c6b2cccbe32c3e497a0ffb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# feature-intensity-pattern-comparison

## Summary

Quantifies how well a re-processed untargeted metabolomics workflow (UmetaFlow/OpenMS) recovers published XCMS/CAMERA features by matching on m/z and retention time and comparing per-sample intensity patterns, per ionization mode. It grounds claims like '90% of published features detected' in reproducible, tolerances-based comparisons.

## When to use

Use when re-processed LC-MS/MS features (m/z, RT, per-sample intensities) must be matched against a published XCMS/CAMERA feature list to compute detection rates and intensity-pattern agreement, e.g., on MetaboLights datasets like MTBLS1129/MTBLS1130.

## When NOT to use

- Input is already a feature table with no raw/mzML data available, since re-quantification and blank/QC handling require re-running the pre-processing on raw files
- There is no published or ground-truth feature list to match against, so no detection-rate or intensity-pattern comparison is possible
- The dataset is GC-MS or MS-free statistics work, outside UmetaFlow's LC-MS/MS scope (Thermo/Agilent/Bruker raw or mzML inputs)

## Inputs

- Centroided mzML files for the public MTBLS1129 and MTBLS1130 colon tumor datasets (Waters UPLC-QTOF), downloaded from MetaboLights
- UmetaFlow configuration (config/config.yaml with ionization modes and re-quantification enabled, config/dataset.tsv, config/blanks.tsv, config/samples.tsv)
- Published XCMS/CAMERA feature lists (m/z, retention time, per-sample intensities) for the same datasets

## Outputs

- detection_rate_comparison.csv reporting fraction of published features detected by UmetaFlow per ionization mode
- UmetaFlow feature matrix per mode (m/z, retention time, per-sample intensities) as TSV
- Feature-intensity correlations across samples comparing UmetaFlow vs published quantification

## How to apply

Run the UmetaFlow Snakemake workflow on the centroided mzML files with feature detection, map alignment, feature grouping, and re-quantification enabled (set re-quantification to TRUE in config/config.yaml) so missing values are re-quantified rather than left as zeros. Export the resulting feature matrix (m/z, retention time, per-sample intensities) separately for positive and negative ionization mode. Match each UmetaFlow feature against the published XCMS/CAMERA feature list using m/z and RT tolerance matching, and compute the fraction of published features detected by UmetaFlow per mode. Additionally correlate feature intensities across samples (e.g., tumor vs normal colon samples in MTBLS1129/MTBLS1130) and compare intensity distributions against the published results. For these datasets the acceptance thresholds are a detection rate of at least 0.90 in positive mode and 1.00 in negative mode, with quantification patterns agreeing with the published XCMS/CAMERA results.

## Related tools

- **UmetaFlow** (Re-detects, aligns, groups and re-quantifies features so their intensities can be compared with the published XCMS/CAMERA feature lists) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS** (Provides the feature detection, map alignment, re-quantification and feature linking algorithms executed by UmetaFlow) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **Snakemake** (Executes the reproducible workflow (dry-run validation, then full run with --use-conda --cores all) that produces the feature matrices to compare) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **pyOpenMS** (Python bindings used in the Jupyter-notebook implementation of the same workflow for exporting and comparing feature tables) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Detection rate on MTBLS1129/MTBLS1130 must be at least 0.90 in positive mode and 1.00 in negative mode after parameter optimization
- Quantification patterns: intensity distributions across tumor vs normal samples must agree with the published XCMS/CAMERA results
- Correlation of feature intensities across samples between UmetaFlow and published data should be consistent, as reported in detection_rate_comparison.csv
- snakemake --use-conda --dry-run must validate the configuration before the full execution
- Feature counts should be consistent with the benchmarking result that UmetaFlow detected more than 90% of all ground truth features on public datasets

## Limitations

- Detection rates depend on prior parameter optimization of the OpenMS/UmetaFlow feature detection for the specific instrument (Waters UPLC-QTOF) and dataset
- The optional minfrac filtering removes consensus features with too many missing values, which can reduce the apparent detection rate if applied too aggressively
- No software version numbers are stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake in the article, complicating exact reproduction
- Benchmark performance numbers (e.g., Tables S5, S7 for MTBLS733/MTBLS736) are only available in supplementary files, not the main text

## Evidence

- [abstract] UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [discussion] detected and quantified with (a) XCMS and (b) UmetaFlow (dataset MTBLS1129): "detected and quantified with (a) XCMS and (b) UmetaFlow (dataset MTBLS1129)"
- [discussion] The optimal parameters for OpenMS (UmetaFlow) for feature detection and quantification of the MTBLS1129 and MTBLS1130 dataset: "The optimal parameters for OpenMS (UmetaFlow) for feature detection and quantification of the MTBLS1129 and MTBLS1130 dataset"
- [intro] using OpenMS 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [readme] Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis: "Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration."
