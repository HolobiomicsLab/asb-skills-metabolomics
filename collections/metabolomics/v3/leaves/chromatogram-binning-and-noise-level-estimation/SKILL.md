---
name: chromatogram-binning-and-noise-level-estimation
description: Use when processing centroided high-resolution LC-MS mzML files where
  you need to convert sparse MS1 spectra into continuous chromatographic mass tracks
  and quantify baseline noise to gate smoothing and peak-detection thresholds.
when_to_use_negative:
- Input data are low resolution — the paper states 'The software does not work with
  low resolution data.'
- Data are of very high noise, which the authors say may need customized processing
- You need MS/MS spectra — 'MS/MS spectra are ignored by asari.' in the default LC-MS
  workflow
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Provides get_thousandth_bins, nn_cluster_by_mz_seeds, extract_single_track_fullrt_length
    and compute_noise_by_flanks for binning tracks and estimating noise
  repo: https://github.com/shuzhao-li-lab/asari
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML that serve as the binning input
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: ProteoWizard converter handling most vendor formats and .mzXML into mzML prior
    to track extraction
  repo: https://proteowizard.sourceforge.io/tools.shtml
- name: Python
  role: Runtime for asari (Python 3.8+) including scipy-based peak detection on the
    binned tracks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:c479b0477c7586cf1372d0229d389022b7f042c008a43d387f18e0acdb668a35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  - msConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Chromatogram binning and noise-level estimation

## Summary

Asari-style discretization of m/z values into 0.001 amu bins to build full-RT mass tracks, plus flank-based noise estimation that decides whether smoothing is applied before statistical elution peak detection.

## When to use

Use when processing centroided high-resolution LC-MS mzML files where you need to convert sparse MS1 spectra into continuous chromatographic mass tracks and quantify baseline noise to gate smoothing and peak-detection thresholds.

## When NOT to use

- Input data are low resolution — the paper states 'The software does not work with low resolution data.'
- Data are of very high noise, which the authors say may need customized processing
- You need MS/MS spectra — 'MS/MS spectra are ignored by asari.' in the default LC-MS workflow

## Inputs

- Centroided mzML file(s) from LC-MS metabolomics acquisitions (MS1 scans)
- min_peak_height parameter (default 1e5) and min_timepoints width parameter

## Outputs

- Per-sample mass tracks: intensity vectors of unique m/z values spanning the full RT range at 0.001 amu m/z granularity
- Per-track noise estimates used to trigger optional smoothing and to set find_peaks prominence
- Detected elution peaks with SNR and peakshape metrics feeding quality filtering (SNR > 2, peakshape > 0.5)

## How to apply

From each centroided mzML file, bin m/z values into 0.001 amu bins (asari's get_thousandth_bins), then NN-cluster bins by m/z using seeds (nn_cluster_by_mz_seeds) to form unique m/z tracks. For each track, extract the intensity vector over the full retention-time range (extract_single_track_fullrt_length) so the chromatogram is a fixed-length vector comparable across samples. Estimate local noise from the flanking regions of candidate peaks (compute_noise_by_flanks); when noise exceeds 1% of the maximum intensity, apply optional smoothing before peak detection. Peak detection is statistics guided: scipy find_peaks on each composite track with dynamically set prominence initialized at 1/3 of the min_peak_height parameter (default 1e5) and width constrained by min_timepoints. The rationale is that mass alignment first, via fine-grained bins, exploits high mass resolution, while explicit noise estimation replaces ad hoc smoothing and keeps every step trackable.

## Related tools

- **asari** (Provides get_thousandth_bins, nn_cluster_by_mz_seeds, extract_single_track_fullrt_length and compute_noise_by_flanks for binning tracks and estimating noise) — https://github.com/shuzhao-li-lab/asari
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML that serve as the binning input) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (ProteoWizard converter handling most vendor formats and .mzXML into mzML prior to track extraction) — https://proteowizard.sourceforge.io/tools.shtml
- **Python** (Runtime for asari (Python 3.8+) including scipy-based peak detection on the binned tracks)

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Binned tracks use 0.001 amu m/z granularity and cover the full RT range as fixed-length intensity vectors
- Noise estimate per track, computed by flanking regions, stays below 1% of max intensity or triggers documented smoothing
- Detected peaks on composite tracks satisfy the quality gates SNR > 2 and peakshape > 0.5
- Peak prominence was set dynamically at 1/3 of the min_peak_height parameter (default 1e5) with width >= min_timepoints
- Exported preferred_Feature_table.tsv contains per-sample peak areas traceable back to composite mass track IDs

## Limitations

- The software does not work with low resolution data
- Data of very high noise may need customized processing
- Efficiency is not optimal on data from longer chromatography

## Evidence

- [methods] From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid.: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
- [readme] Statistics guided peak detection on local maxima, prominence and selective smoothing: "Statistics guided peak dection, based on local maxima and prominence, selective use of smoothing"
- [discussion] The software does not work with low resolution data.: "The software does not work with low resolution data."
- [discussion] data of very high noise may need customized processing: "data of very high noise may need customized processing"
- [readme] A typical run of data processing may generatae a directory ... The recommended feature table is preferred_Feature_table.tsv.: "The recommended feature table is `preferred_Feature_table.tsv`."
