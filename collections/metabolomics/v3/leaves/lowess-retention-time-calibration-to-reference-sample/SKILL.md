---
name: lowess-retention-time-calibration-to-reference-sample
description: Use when aligned per-sample mass tracks (from centroided mzML files)
  show retention time drift across samples in a study and must be mapped to a common
  reference sample before aggregating into composite mass tracks.
when_to_use_negative:
- Input data are low-resolution mass spectra — asari's mass-track framework (and its
  RT calibration step) is built for high-resolution data and 'does not work with low
  resolution data'.
- Data have very high noise or come from long chromatography runs — the article notes
  'data of very high noise may need customized processing' and efficiency 'is not
  optimal on data from longer chromatography'.
- The goal is MS/MS-based analysis — 'MS/MS spectra are ignored by asari' in the default
  LC-MS workflow, so RT calibration of MS1 mass tracks alone will not serve MS/MS-centric
  pipelines.
tools:
- name: asari
  role: Implements the LOWESS retention time calibration as chromatograms.rt_lowess_calibration
    within its mass-track processing pipeline.
  repo: https://github.com/shuzhao-li/asari
- name: ThermoRawFileParser
  role: Upstream conversion of Thermo .RAW files to the centroided .mzML files that
    feed the mass-track extraction and calibration step.
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:90b95832f19fa72294141c9f51439a2211472ff36f71d4b641f50ac07202bfff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  - msConvert
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lowess-retention-time-calibration-to-reference-sample

## Summary

Calibrates retention time of each LC-MS sample to a common reference sample via LOWESS regression on matched landmark elution peaks, so per-sample mass tracks can be summed into composite mass tracks in asari. This corrects chromatographic drift before peak detection is performed once per composite track rather than per sample.

## When to use

Use when aligned per-sample mass tracks (from centroided mzML files) show retention time drift across samples in a study and must be mapped to a common reference sample before aggregating into composite mass tracks.

## When NOT to use

- Input data are low-resolution mass spectra — asari's mass-track framework (and its RT calibration step) is built for high-resolution data and 'does not work with low resolution data'.
- Data have very high noise or come from long chromatography runs — the article notes 'data of very high noise may need customized processing' and efficiency 'is not optimal on data from longer chromatography'.
- The goal is MS/MS-based analysis — 'MS/MS spectra are ignored by asari' in the default LC-MS workflow, so RT calibration of MS1 mass tracks alone will not serve MS/MS-centric pipelines.

## Inputs

- Centroided mzML files from an LC-MS metabolomics study
- Per-sample mass tracks extracted over the full RT range (intensity vectors per m/z bin)
- MassGrid of mass tracks aligned across samples
- Reference sample selected by the highest number of anchor mass tracks, with its landmark elution peaks

## Outputs

- rt_cal_dict mapping each sample's scan numbers to reference-sample scan numbers (with 10% extension at both ends)
- RT-aligned per-sample mass tracks ready for summation into composite mass tracks (CompositeMap)
- Per-sample peak areas recovered by translating composite-track peak boundaries back through the RT calibration dictionaries

## How to apply

In asari, aligned mass tracks are first arranged into a MassGrid; select the reference sample by the highest number of anchor mass tracks, then identify landmark elution peaks restricted to high-quality features (mSelectivity > 0.99, peak height at least 1e5, prominence above 20% of peak height, and a single peak per track). For every other sample, pick the matching good landmark peaks and fit a LOWESS regression between the sample's and the reference's scan numbers, adding a 10% extension at both ends of the range to build the rt_cal_dict mapping. Apply rt_cal_dict to RT-align each sample's mass tracks, then sum the aligned tracks into composite mass tracks in the CompositeMap; after peak detection on composite tracks, the same dictionaries are used to translate composite-track peak boundaries back to per-sample peak areas. The implementation lives in the chromatograms module as rt_lowess_calibration, and retention time alignment can alternatively take advantage of spike-in standards when available.

## Related tools

- **asari** (Implements the LOWESS retention time calibration as chromatograms.rt_lowess_calibration within its mass-track processing pipeline.) — https://github.com/shuzhao-li/asari
- **ThermoRawFileParser** (Upstream conversion of Thermo .RAW files to the centroided .mzML files that feed the mass-track extraction and calibration step.) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python3 -m asari.main process -i mydir/projectx_dir; from asari.chromatograms import rt_lowess_calibration
```

## Evaluation signals

- The fitted rt_cal_dict covers the full RT range including the 10% extension at both ends, so no sample scans fall outside the calibration mapping.
- All landmarks used for the LOWESS fit pass the stated quality gates: mSelectivity > 0.99, peak height ≥ 1e5, prominence > 20% of peak height, and exactly one peak per track.
- After calibration, matched landmark peaks across samples align in scan number against the reference sample before tracks are summed into composite mass tracks.
- The exported preferred_Feature_table.tsv contains per-sample peak areas whose values are traceable back to composite mass track IDs via the RT calibration dictionaries.
- A downstream run of the full pipeline completes and produces the expected output directory (preferred_Feature_table.tsv, export/full_Feature_table.tsv, project.json).

## Limitations

- Calibration quality depends on the supply of high-quality landmark peaks (mSelectivity > 0.99, height ≥ 1e5); samples lacking enough matched landmarks will calibrate poorly.
- The article notes additional methods of retention time alignment could take advantage of spike-in standards, implying the landmark-based LOWESS approach does not use internal standards.
- Asari has mostly been tested on Orbitrap platforms; settings may need further optimization for other acquisition platforms, which affects the landmark peak thresholds.

## Evidence

- [methods] Retention time is calibrated for each sample to a common reference sample.: "Retention time is calibrated for each sample to a common reference sample."
- [methods] [chromatograms.rt_lowess_calibration](chromatograms.rt_lowess_calibration): "[chromatograms.rt_lowess_calibration](chromatograms.rt_lowess_calibration)"
- [discussion] additional methods of retention time alignment can take advantage of spike-in standards: "additional methods of retention time alignment can take advantage of spike-in standards"
- [readme] Reproducible, track and backtrack between features and mass tracks (EICs): "Reproducible, track and backtrack between features and mass tracks (EICs)"
