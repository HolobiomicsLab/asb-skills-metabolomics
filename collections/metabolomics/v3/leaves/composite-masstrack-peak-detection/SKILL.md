---
name: composite-masstrack-peak-detection
description: Use when processing multiple centroided LC-MS metabolomics mzML files
  where features must be detected reproducibly across samples and each feature must
  be traceable back to its underlying mass track (EIC).
when_to_use_negative:
- Input is low-resolution MS data — the software does not work with low resolution
  data
- Data are very high noise or from long chromatography gradients — efficiency is not
  optimal and customized processing may be needed
- Input is already a processed feature table (asari is designed to separate annotation/feature
  extraction; use khipu or annotation workflows instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: asari
  role: Implements mass-track extraction, MassGrid alignment, LOWESS RT calibration,
    composite track summation and peak detection
  repo: https://github.com/shuzhao-li-lab/asari
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to .mzML before mass-track building
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: Alternative converter for most vendor data formats and .mzXML into centroided
    mzML
  repo: https://proteowizard.sourceforge.io/tools.shtml
- name: Python
  role: Runtime for asari (requires Python 3.8+); scipy find_peaks used for elution
    peak detection on composite tracks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:f5d7b2971aa12ab99c75ad5cfc80256ad3b78ad3a6c4ede98cbf59a300e1522a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  - msConvert
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# composite-masstrack-peak-detection

## Summary

Asari's approach of RT-aligning per-sample mass tracks into composite mass tracks and running elution peak detection once on the composite map, rather than detecting peaks repeatedly in individual samples, improving consistency, trackability, and computational performance.

## When to use

Use when processing multiple centroided LC-MS metabolomics mzML files where features must be detected reproducibly across samples and each feature must be traceable back to its underlying mass track (EIC).

## When NOT to use

- Input is low-resolution MS data — the software does not work with low resolution data
- Data are very high noise or from long chromatography gradients — efficiency is not optimal and customized processing may be needed
- Input is already a processed feature table (asari is designed to separate annotation/feature extraction; use khipu or annotation workflows instead)

## Inputs

- Centroided mzML files from multiple LC-MS samples in one directory
- Optional converted raw data (Thermo .RAW converted via ThermoRawFileParser or msConvert)
- Processing parameters (mode pos/neg, min_peak_height, default 1e5)

## Outputs

- CompositeMap of composite mass tracks (export/cmap.pickle)
- preferred_Feature_table.tsv with per-sample peak areas and links to composite mass track IDs
- export/full_Feature_table.tsv retaining all peaks meeting snr and shape standards
- project.json and pickle files for intermediate sample tracks

## How to apply

First build per-sample mass tracks by binning m/z into 0.001 amu bins, NN-clustering bins by m/z using seeds, and extracting each track over the full RT range while prioritizing 13C/12C isotopic and Na/H adduct anchor pairs. Align tracks across samples into a MassGrid, mapping anchor mass tracks and recalibrating m/z when a systematic difference above 1 ppm is detected. Select the reference sample with the most anchor mass tracks, identify landmark elution peaks (mSelectivity > 0.99, height ≥ 1e5, prominence > 20% of peak height, single peak per track), and for each other sample fit a LOWESS regression between sample and reference scan numbers (10% extension at both ends) to build the rt_cal_dict. Then sum RT-aligned per-sample tracks into composite tracks in the CompositeMap and detect elution peaks once per composite track with scipy find_peaks using dynamically set prominence (initial 1/3 of the 1e5 min_peak_height), min_timepoints for width, and optional smoothing when noise exceeds 1% of max intensity. Finally evaluate peaks for peakshape and cSelectivity, keep peaks passing SNR > 2 and peakshape > 0.5, and translate composite-track peak boundaries back to each sample via the RT calibration dictionaries to record per-sample peak areas.

## Related tools

- **asari** (Implements mass-track extraction, MassGrid alignment, LOWESS RT calibration, composite track summation and peak detection) — https://github.com/shuzhao-li-lab/asari
- **ThermoRawFileParser** (Converts Thermo .RAW files to .mzML before mass-track building) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (Alternative converter for most vendor data formats and .mzXML into centroided mzML) — https://proteowizard.sourceforge.io/tools.shtml
- **Python** (Runtime for asari (requires Python 3.8+); scipy find_peaks used for elution peak detection on composite tracks)

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Detected peaks pass SNR > 2 and peakshape > 0.5 filters
- Landmark peaks used for RT calibration satisfy mSelectivity > 0.99, peak height ≥ 1e5, and prominence above 20% of peak height
- preferred_Feature_table.tsv exists with per-sample peak areas and each feature is trackable back to its composite mass track ID
- Exported feature table compares favorably to XCMS and MZmine in feature detection and quantification
- Composite track peak boundaries can be translated back to every sample via rt_cal_dict without unmatched samples

## Limitations

- MS/MS spectra are ignored in the default LC-MS workflow
- Settings are mostly tested on Orbitrap platforms and need optimization elsewhere; low-resolution data are unsupported
- A 2nd order polynomial model may be required for accurate mass calibration across the full m/z range since ppm is only a practical approximation
- Borderline peak filtering cases involve a tradeoff between sensitivity and true discovery rate

## Evidence

- [methods] Composite tracks summarize per-sample tracks after RT calibration: "For each m/z value, corresponding mass tracks from all sample files are summarized into one composite mass track."
- [methods] RT calibration is anchored on a common reference sample: "Retention time is calibrated for each sample to a common reference sample."
- [readme] Peak detection runs once on the composite map, not per sample: "Peak detection on a composite map instead of repeated on individual samples"
- [readme] Peak detection is statistics-guided via local maxima and prominence: "Statistics guided peak dection, based on local maxima and prominence, selective use of smoothing"
- [methods] Mass tracks are extracted per file then aligned into a MassGrid: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
