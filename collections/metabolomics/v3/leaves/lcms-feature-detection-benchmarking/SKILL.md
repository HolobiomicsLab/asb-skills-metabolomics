---
name: lcms-feature-detection-benchmarking
description: Use when multiple LC-MS data processing tools must be compared for feature
  detection consistency, or when reported feature counts (e.g. 10,901 XCMS vs 42,099
  MZmine features on 184 pooled HZV029 plasma samples) need to be reproduced.
when_to_use_negative:
- Input data are low-resolution MS or from non-Orbitrap platforms — the article notes
  asari does not work with low-resolution data and its settings need optimization
  for other platforms
- The goal is MS/MS-based annotation from the raw runs — asari's default LC-MS workflow
  ignores MS/MS spectra
- Input is already a processed feature table rather than raw centroided mzML — feature
  detection comparison requires reprocessing from the same mzML files
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Open-source Python tool for trackable, scalable LC-MS feature detection used
    as the benchmark subject; processes centroided mzML into preferred_Feature_table.tsv
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Widely used comparison tool run with the paper's peak-picking and correspondence
    parameters to generate a feature table whose row count is benchmarked
- name: MZmine
  role: Widely used comparison tool run with the paper's peak detection and deconvolution
    modules to produce the second feature table for correspondence analysis
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to .mzML so all tools process identical centroided
    inputs
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: ProteoWizard converter for non-Thermo vendor formats and .mzXML into mzML
    prior to processing
  repo: https://proteowizard.sourceforge.io/tools.shtml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:23669b8c942c8e035a72c3001cf19cb8c976b3a2bcf54034057bd1b226273698
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  - msConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-feature-detection-benchmarking

## Summary

A protocol for benchmarking LC-MS feature detection and correspondence across metabolomics processing tools (XCMS, MZmine, asari) by running each tool over the same centroided mzML dataset and comparing feature counts, cross-tool matching rates, and computational performance against published reference values.

## When to use

Use when multiple LC-MS data processing tools must be compared for feature detection consistency, or when reported feature counts (e.g. 10,901 XCMS vs 42,099 MZmine features on 184 pooled HZV029 plasma samples) need to be reproduced.

## When NOT to use

- Input data are low-resolution MS or from non-Orbitrap platforms — the article notes asari does not work with low-resolution data and its settings need optimization for other platforms
- The goal is MS/MS-based annotation from the raw runs — asari's default LC-MS workflow ignores MS/MS spectra
- Input is already a processed feature table rather than raw centroided mzML — feature detection comparison requires reprocessing from the same mzML files

## Inputs

- 184 pooled human plasma LC-MS mzML files from the HZV029 study (centroided, from Thermo .RAW via ThermoRawFileParser or msConvert)
- Vendor RAW files from Thermo instruments (pre-conversion)
- Peak-picking and correspondence parameters used in the reference paper for XCMS and MZmine

## Outputs

- Per-tool feature tables (e.g. XCMS and MZmine feature rows; asari preferred_Feature_table.tsv / full_Feature_table.tsv)
- Summary table of detected feature counts per tool (expected reference values: 10,901 XCMS features and 42,099 MZmine features)
- Cross-tool correspondence/matching rate (reference: ~60% of XCMS features unambiguously matched to MZmine features)

## How to apply

Obtain the raw LC-MS runs for the same sample set (e.g. 184 pooled human plasma HZV029 files), convert vendor RAW files to centroided mzML with ThermoRawFileParser or msConvert, then process the identical dataset with each tool using comparable peak-picking and correspondence parameters (XCMS and MZmine in the reference study). For each tool, count feature rows in the resulting feature table and record them in a summary table, then quantify cross-tool correspondence by matching features on m/z and retention time (the reference study found only ~60% of XCMS features unambiguously matched to MZmine). Optionally run asari over the same mzML inputs — its mass-alignment-first design (mass tracks into a MassGrid, composite mass tracks across samples) is explicitly trackable and enables backtracking features to mass tracks. Judge correctness by confirming feature counts against published reference values (allowing minor deviation for software version drift) and by inspecting feature-to-EIC traceability via asari viz.

## Related tools

- **asari** (Open-source Python tool for trackable, scalable LC-MS feature detection used as the benchmark subject; processes centroided mzML into preferred_Feature_table.tsv) — https://github.com/shuzhao-li/asari
- **XCMS** (Widely used comparison tool run with the paper's peak-picking and correspondence parameters to generate a feature table whose row count is benchmarked)
- **MZmine** (Widely used comparison tool run with the paper's peak detection and deconvolution modules to produce the second feature table for correspondence analysis)
- **ThermoRawFileParser** (Converts Thermo .RAW files to .mzML so all tools process identical centroided inputs) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (ProteoWizard converter for non-Thermo vendor formats and .mzXML into mzML prior to processing) — https://proteowizard.sourceforge.io/tools.shtml

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Feature counts per tool match the published reference values (10,901 XCMS features; 42,099 MZmine features), with minor deviation acceptable due to software version drift
- Cross-tool correspondence rate is reported as the fraction of XCMS features unambiguously matched to MZmine features (reference: ~60%)
- asari run produces a complete project directory including preferred_Feature_table.tsv and export/full_Feature_table.tsv, with every feature traceable back to its mass track/EIC
- All tools processed the identical set of centroided mzML files converted from the same RAW files
- Inconsistency observed across tools should be explainable via mass alignment and feature quality control differences, per the paper's attribution

## Limitations

- Software version drift between the paper's versions and current releases causes minor deviation from reported feature counts
- asari settings are not optimized for non-Orbitrap platforms and it does not work with low-resolution data
- Efficiency is not optimal on data from longer chromatography, and very high noise data may need customized processing
- Borderline feature cases trade off sensitivity versus true discovery rate, affecting cross-tool matching

## Evidence

- [intro] Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7.: "Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7."
- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [methods] We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
- [readme] Input data are centroid mzML files from LC, GC or DI-MS metabolomics.: [evidence span withheld — anchor status: external_doc]
- [discussion] In this study, we attribute the inconsistency in previous software tools to poor mass alignment and a lack of quality metrics.: "In this study, we attribute the inconsistency in previous software tools to poor mass alignment and a lack of quality metrics."
