---
name: local-maxima-peak-detection-parameter-matching
description: Use when validating or reimplementing asari's chromatographic peak calling
  on composite mass tracks, e.g. to recompute snr and cSelectivity from export/cmap.pickle
  and compare against full_Feature_table.tsv values.
when_to_use_negative:
- Input is already an asari preferred_Feature_table.tsv and you only need reported
  quality metrics — the exported snr, cSelectivity, and peakshape columns can be used
  directly without recomputation
- Data are low-resolution or non-Orbitrap platforms where asari's settings are explicitly
  not yet optimized and the min_peak_height=1e5 default does not apply
- You need de novo peak detection on raw vendor files (Thermo .RAW); asari requires
  centroided mzML input, so conversion via ThermoRawFileParser/msConvert must happen
  first
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: Source of the peak-detection functions being matched (peaks.stats_detect_elution_peaks,
    compute_noise_by_flanks, __peaks_cSelectivity_stats_) and of the exported feature
    tables and cmap pickle used for validation
  repo: https://github.com/shuzhao-li/asari
- name: Python
  role: Implementation language for recomputing snr and cSelectivity on mass-track
    intensity vectors (asari requires Python 3.8+)
- name: ThermoRawFileParser
  role: Upstream converter of Thermo .RAW files to centroided .mzML that asari ingests
    before mass tracks are built
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:a73b634b77f7822121187bbd0f0149301d815bf624b90c9b5d0f7c8cd0e46996
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# local-maxima-peak-detection-parameter-matching

## Summary

Reproduces asari's elution-peak detection on mass tracks (EICs) by matching the parameters used in peaks.stats_detect_elution_peaks — prominence derived from min_peak_height and track noise, plus min_timepoints-based distance and width constraints — so that independently computed peak regions, SNR, and cSelectivity align with asari-exported feature-table values.

## When to use

Use when validating or reimplementing asari's chromatographic peak calling on composite mass tracks, e.g. to recompute snr and cSelectivity from export/cmap.pickle and compare against full_Feature_table.tsv values.

## When NOT to use

- Input is already an asari preferred_Feature_table.tsv and you only need reported quality metrics — the exported snr, cSelectivity, and peakshape columns can be used directly without recomputation
- Data are low-resolution or non-Orbitrap platforms where asari's settings are explicitly not yet optimized and the min_peak_height=1e5 default does not apply
- You need de novo peak detection on raw vendor files (Thermo .RAW); asari requires centroided mzML input, so conversion via ThermoRawFileParser/msConvert must happen first

## Inputs

- asari project export/cmap.pickle (dict of composite mass track intensity vectors keyed by parent_masstrack_id)
- export/full_Feature_table.tsv with peak height, left_base/right_base boundaries, parent_masstrack_id, and exported snr/cSelectivity per feature
- asari processing parameters, notably min_peak_height (default 1e5 for Orbitrap) and min_timepoints

## Outputs

- Per-feature recomputed SNR (peak height / mean flank noise, up to 100 non-peak points per side)
- Per-feature recomputed cSelectivity in [0, 1] (fraction of above-half-height points belonging to the peak)
- Comparison table of recomputed vs asari-exported snr and cSelectivity values

## How to apply

Retrieve the composite mass track intensity vector for a feature via its parent_masstrack_id from export/cmap.pickle and slice the region bounded by left_base and right_base in full_Feature_table.tsv. Run peak detection on the track as asari does in peaks.stats_detect_elution_peaks: initialize scipy-style prominence to one-third of min_peak_height (default 1e5 for Orbitrap data), raise it to at least the track's noise level, and apply min_timepoints-based distance and width constraints so detected peak regions match those the software itself called. Then compute SNR as peak height divided by the mean intensity of up to 100 non-peak flank points per side (mirroring peaks.compute_noise_by_flanks), and cSelectivity as the fraction of points above half the peak height that belong to the evaluated peak (per peaks.evaluate_gaussian_peak_on_intensity_list and peaks.__peaks_cSelectivity_stats_). The rationale is that matching the exact detection parameters is a prerequisite — without them the peak boundaries differ and recomputed metrics cannot be compared to exported ones, as in Fig. 6a-b.

## Related tools

- **asari** (Source of the peak-detection functions being matched (peaks.stats_detect_elution_peaks, compute_noise_by_flanks, __peaks_cSelectivity_stats_) and of the exported feature tables and cmap pickle used for validation) — https://github.com/shuzhao-li/asari
- **Python** (Implementation language for recomputing snr and cSelectivity on mass-track intensity vectors (asari requires Python 3.8+))
- **ThermoRawFileParser** (Upstream converter of Thermo .RAW files to centroided .mzML that asari ingests before mass tracks are built) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process -i MT202304_2phase_HILICpos -o highonly --min_peak_height 1000000
```

## Evaluation signals

- Recomputed SNR matches the asari-exported snr for the same features within acceptable tolerance (as in Fig. 6a-b)
- Recomputed cSelectivity matches exported values; exported cSelectivity ranges up to 1.0 (e.g. 1.0 for the F122/F46 peaks in the documented empirical compound example)
- Peaks passing asari's own filters report snr > 2 and peakshape > 0.5 in the exported feature table
- Detected peak boundaries in the recomputed run coincide with the left_base/right_base boundaries recorded in full_Feature_table.tsv
- Full export only retains peaks meeting signal (snr) and shape standards, so every feature present in full_Feature_table.tsv should pass the recomputed thresholds

## Limitations

- Asari's settings are mostly tested on Orbitrap platforms; min_peak_height 1e5 and related defaults need optimization for other instruments
- Asari does not work with low-resolution data, and very high-noise data may need customized processing
- Efficiency is not optimal for longer chromatography data; borderline peak calls are a tradeoff between sensitivity and true discovery rate

## Evidence

- [methods] prominence initialized to one-third of min_peak_height (default 1e5 for Orbitrap data), raised to at least the track noise level, plus min_timepoints-based distance and width constraints: [evidence span withheld — anchor status: not_found]
- [results] SNR is defined as peak height divided by the noise level, where noise is the average intensity of up to 100 non-peak data points on each side of the peak: [evidence span withheld — anchor status: not_found]
- [results] Asari computes cSelectivity after filtering the chromatogram at half the peak height, as the fraction of data points belonging to any peak: [evidence span withheld — anchor status: not_found]
- [readme] All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
- [readme] Statistics guided peak dection, based on local maxima and prominence, selective use of smoothing: "Statistics guided peak dection, based on local maxima and prominence, selective use of smoothing"
