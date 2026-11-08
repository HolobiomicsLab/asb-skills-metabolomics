---
name: mass-spectrometry-data-preprocessing
description: Use when raw vendor LC-MS files (Thermo .RAW, or other vendor formats/.mzXML)
  from a high-resolution instrument must be turned into centroided mzML and a cross-sample
  LC-MS feature table, or when reproducible, trackable feature detection is required.
when_to_use_negative:
- Input is low-resolution mass spectrometry data — the paper states 'The software
  does not work with low resolution data.'
- The goal is MS/MS-centric analysis with the default LC-MS workflow — 'MS/MS spectra
  are ignored by asari' in the default path (use alternative/PCPFM workflows instead).
- Data are already processed into a feature table and only downstream annotation or
  statistics are needed — use khipu/mummichog or the annotation subcommands instead.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: asari
  role: 'Core preprocessing engine: mass alignment into MassGrid, RT calibration,
    composite mass track peak detection, and export of trackable feature tables'
  repo: https://github.com/shuzhao-li/asari
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML as the input-preparation step
    before asari processing
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: ProteoWizard tool for converting most vendor data formats and .mzXML to centroided
    mzML
  repo: https://proteowizard.sourceforge.io/tools.shtml
- name: XCMS
  role: Widely used alternative processor used as a benchmark for feature detection
    and correspondence consistency
- name: MZmine
  role: Widely used alternative processor used as a benchmark for feature detection
    and correspondence consistency
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:f462f0c6ac600fe998cd63701ddfb3f955236a156b33812ddc08d5919c81c11b
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mass-spectrometry-data-preprocessing

## Summary

Preprocess raw LC-MS metabolomics acquisitions into centroided mzML and a trackable feature table: vendor .RAW files are converted/centroided with ThermoRawFileParser or msConvert, then processed with asari's mass-alignment-first framework (mass tracks in a MassGrid, RT calibration, composite mass tracks, peak detection on the composite map).

## When to use

Use when raw vendor LC-MS files (Thermo .RAW, or other vendor formats/.mzXML) from a high-resolution instrument must be turned into centroided mzML and a cross-sample LC-MS feature table, or when reproducible, trackable feature detection is required.

## When NOT to use

- Input is low-resolution mass spectrometry data — the paper states 'The software does not work with low resolution data.'
- The goal is MS/MS-centric analysis with the default LC-MS workflow — 'MS/MS spectra are ignored by asari' in the default path (use alternative/PCPFM workflows instead).
- Data are already processed into a feature table and only downstream annotation or statistics are needed — use khipu/mummichog or the annotation subcommands instead.

## Inputs

- Raw vendor LC-MS data (Thermo .RAW files)
- Centroided mzML files (required input for the asari default LC-MS workflow)
- Other vendor formats and .mzXML convertible via msConvert (ProteoWizard)

## Outputs

- preferred_Feature_table.tsv (recommended cross-sample feature table with m/z, retention time, per-sample intensities)
- export/full_Feature_table.tsv (all peaks meeting snr and shape standards, including single-sample features)
- export/_mass_grid_mapping.csv and composite mass track data (cmap.pickle)
- project.json and per-sample pickle intermediates supporting track-and-backtrack between features and mass tracks

## How to apply

First convert Thermo .RAW files to centroided .mzML with ThermoRawFileParser (or msConvert for other vendor formats and .mzXML); asari's default LC-MS workflow requires centroided mzML as input. Run asari on the directory of mzML files in the desired ionization mode; asari extracts mass tracks of unique m/z values from each file, aligns them into a MassGrid, calibrates retention time for each sample to a common reference sample, and summarizes corresponding mass tracks across all samples into composite mass tracks on which peak detection is performed once, instead of repeatedly per sample. This mass-alignment-first design exploits high mass resolution and avoids the correspondence errors the paper attributes to poor mass alignment and lack of quality metrics in tools like XCMS and MZmine. Peaks are detected using local maxima and prominence statistics, with selective use of smoothing; per-feature quality is tracked via snr and shape/selectivity metrics so borderline cases are left visible to the user rather than silently dropped. The recommended end output is preferred_Feature_table.tsv, while full_Feature_table.tsv retains all peaks meeting the snr and shape standards (e.g., even single-sample features), leaving filtering decisions to the end user.

## Related tools

- **asari** (Core preprocessing engine: mass alignment into MassGrid, RT calibration, composite mass track peak detection, and export of trackable feature tables) — https://github.com/shuzhao-li/asari
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML as the input-preparation step before asari processing) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (ProteoWizard tool for converting most vendor data formats and .mzXML to centroided mzML) — https://proteowizard.sourceforge.io/tools.shtml
- **XCMS** (Widely used alternative processor used as a benchmark for feature detection and correspondence consistency)
- **MZmine** (Widely used alternative processor used as a benchmark for feature detection and correspondence consistency)

## Examples

```
ThermoRawFileParser --input raw_dir --output mzml_dir --vendor Thermo; asari process --mode pos --input mzml_dir
```

## Evaluation signals

- Feature counts on a shared benchmark dataset should be of the same order as published processor outputs — on the 184-sample HZV029 pooled plasma data, XCMS reported 10,901 features vs MZmine's 42,099, so a large discrepancy signals correspondence instability to check
- Every reported feature must backtrack to its underlying mass track (EIC) — 'Reproducible, track and backtrack between features and mass tracks (EICs)' per the README
- full_Feature_table.tsv should include features meeting the configured signal-to-noise (snr) and shape standards, including single-sample features; preferred_Feature_table.tsv is the QC-filtered subset
- Run 'asari analyze' on a single mzML file first and confirm the returned per-file summary (scan-level statistics) looks consistent with the platform before batch processing
- The exported project folder must contain project.json, export/cmap.pickle, export/epd.pickle and export/full_Feature_table.tsv so the dashboard can render EICs for visual verification

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings need further optimization for other instruments
- The software does not work with low-resolution data
- Efficiency is not optimal on data from longer chromatography
- Data of very high noise may need customized processing
- A 2nd order polynomial m/z model may be required to cover the full m/z range more accurately than ppm-based approximation
- MS/MS spectra are ignored by the default LC-MS workflow

## Evidence

- [methods] ThermoRawFileParser converts RAW to mzML: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
- [methods] Mass tracks extracted and aligned into MassGrid: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
- [methods] RT calibration to common reference: "Retention time is calibrated for each sample to a common reference sample."
- [intro] Correspondence challenge in cross-sample alignment: "When peaks are later aligned cross samples (i.e., correspondence), the m/z values vary slightly in each sample and the algorithm must ensure"
- [abstract] Inconsistency attributed to mass alignment and quality control: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [readme] README: required input format: [evidence span withheld — anchor status: external_doc]
- [readme] README: full feature table retains low-support peaks: "if a feature is only present in one sample, it will be reported, 
as we think this is important for applications like exposome and personalized medicine."
