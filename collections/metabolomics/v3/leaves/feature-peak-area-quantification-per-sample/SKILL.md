---
name: feature-peak-area-quantification-per-sample
description: Use when processing centroided LC-MS mzML metabolomics data and you need
  per-sample quantitative peak areas for features that were detected only once on
  composite (cross-sample) mass tracks, with traceability back to mass tracks.
when_to_use_negative:
- Input is low-resolution mass spectrometry data — asari exploits high mass resolution
  and does not work with low resolution data
- Input is already a processed feature table (e.g. you only need annotation or QC
  downstream — use asari annotate or mummichog instead)
- Data are very high noise or from long chromatography, where the article notes efficiency
  is suboptimal and customized processing may be needed
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: asari
  role: Performs composite-track peak detection (scipy find_peaks with dynamic prominence),
    computes per-sample peak areas via rt_cal_dict boundary translation, and writes
    preferred_Feature_table.tsv and full_Feature_table.tsv.
  repo: https://github.com/shuzhao-li-lab/asari
- name: Python
  role: Runtime required to run asari (Python 3.8+); peak detection and quantification
    logic is implemented in Python (e.g. peaks.stats_detect_elution_peaks).
  repo: https://github.com/shuzhao-li-lab/asari
- name: ThermoRawFileParser
  role: Upstream conversion of Thermo .RAW files to the centroided .mzML inputs that
    asari's quantification workflow consumes.
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:6edbdeee61abc7a8432faee2ff5551bad47dabdec5f06dce127626f2bb6fde81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  - msConvert
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# feature-peak-area-quantification-per-sample

## Summary

Asari's approach to per-sample quantification: elution peaks are detected once on composite mass tracks (summed across samples), then peak boundaries are translated back into each sample using the per-sample LOWESS RT calibration dictionaries to record per-sample peak areas in a trackable feature table.

## When to use

Use when processing centroided LC-MS mzML metabolomics data and you need per-sample quantitative peak areas for features that were detected only once on composite (cross-sample) mass tracks, with traceability back to mass tracks.

## When NOT to use

- Input is low-resolution mass spectrometry data — asari exploits high mass resolution and does not work with low resolution data
- Input is already a processed feature table (e.g. you only need annotation or QC downstream — use asari annotate or mummichog instead)
- Data are very high noise or from long chromatography, where the article notes efficiency is suboptimal and customized processing may be needed

## Inputs

- Composite mass tracks stored in the CompositeMap (summed from RT-aligned per-sample mass tracks)
- Per-sample rt_cal_dict RT calibration mappings built by LOWESS regression against the reference sample
- Centroided mzML files per sample (converted from Thermo .RAW via ThermoRawFileParser or msConvert)
- Processing parameters: min_peak_height (default 1e5), SNR and peakshape thresholds

## Outputs

- preferred_Feature_table.tsv in the project directory with per-sample peak areas per feature
- export/full_Feature_table.tsv with all peaks meeting signal (snr) and shape standards
- Trackable links from each feature row to composite mass track IDs
- Project JSON data structures (project.json, export/cmap.pickle)

## How to apply

After aligned per-sample mass tracks are RT-calibrated (LOWESS regression on landmark elution peaks, mSelectivity > 0.99, peak height ≥ 1e5, prominence > 20% of peak height) and summed into composite mass tracks in the CompositeMap, detect elution peaks once per composite track using scipy find_peaks with dynamic prominence (initial 1/3 of the 1e5 min_peak_height), min_timepoints width constraint, and optional smoothing when noise exceeds 1% of max intensity. Evaluate each detected peak for peakshape, cSelectivity and SNR, keeping peaks passing SNR > 2 and peakshape > 0.5. For kept peaks, translate composite-track peak boundaries back to each sample via the rt_cal_dict mapping to record per-sample peak areas. Export the feature table as preferred_Feature_table.tsv; keep all peaks meeting signal (snr) and shape standards in export/full_Feature_table.tsv, leaving filtering decisions to end users.

## Related tools

- **asari** (Performs composite-track peak detection (scipy find_peaks with dynamic prominence), computes per-sample peak areas via rt_cal_dict boundary translation, and writes preferred_Feature_table.tsv and full_Feature_table.tsv.) — https://github.com/shuzhao-li-lab/asari
- **Python** (Runtime required to run asari (Python 3.8+); peak detection and quantification logic is implemented in Python (e.g. peaks.stats_detect_elution_peaks).) — https://github.com/shuzhao-li-lab/asari
- **ThermoRawFileParser** (Upstream conversion of Thermo .RAW files to the centroided .mzML inputs that asari's quantification workflow consumes.) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- preferred_Feature_table.tsv exists in the output directory with per-sample peak area columns for each feature
- Each feature row is trackable/backtrackable to a composite mass track ID (e.g. via export/_mass_grid_mapping.csv and cmap.pickle)
- Retained features satisfy the default quality gates: SNR > 2 and peakshape > 0.5; full_Feature_table.tsv contains all peaks meeting snr/shape standards including single-sample features
- Peak detection used dynamic prominence initialized at 1/3 of the min_peak_height (1e5 default) on composite tracks, and peak boundaries map back to per-sample scans via the RT calibration dictionaries without gaps for samples lacking landmark peaks (10% LOWESS extension at both ends)
- As a paper-level check, feature detection and quantification compare favorably to XCMS and MZmine

## Limitations

- MS/MS spectra are ignored by asari in the default LC-MS workflow (alternative workflows handle them)
- Settings are mostly tested on Orbitrap platforms and need optimization for others; low-resolution data are not supported
- Efficiency is not optimal on data from longer chromatography, and very high noise data may need customized processing
- Filtering of weak/single-sample features is deliberately left to end users rather than hard-coded

## Evidence

- [methods] composite mass track summarization: "For each m/z value, corresponding mass tracks from all sample files are summarized into one composite mass track."
- [methods] RT calibration to reference sample: "Retention time is calibrated for each sample to a common reference sample."
- [readme] full feature table retention of single-sample features: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
- [readme] recommended output table: "The recommended feature table is `preferred_Feature_table.tsv`."
- [abstract] quantification performance claim: "Asari compares favorably to other tools in feature detection and quantification"
