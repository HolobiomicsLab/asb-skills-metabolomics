---
name: peak-cselectivity-fraction-calculation
description: Use when validating or re-implementing asari peak quality metrics on
  extracted ion chromatograms, or when assessing whether a chromatographic peak in
  a composite mass track is selective (cSelectivity up to 1.0) versus contaminated
  by co-eluting signal.
when_to_use_negative:
- Input is a low-resolution or very high noise LC-MS dataset — asari is mostly tested
  on high-resolution Orbitrap platforms and does not work with low resolution data
- You only need the exported quality values — cSelectivity and snr are already carried
  in full_Feature_table.tsv and can be filtered directly (snr > 2, peakshape > 0.5)
- Analysis targets MS/MS spectra — these are ignored by asari's default LC-MS workflow
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Source of the cSelectivity definition and exporter of full_Feature_table.tsv
    and cmap.pickle used to compute and validate the metric
  repo: https://github.com/shuzhao-li/asari
- name: Python
  role: Implementation language for recomputing cSelectivity from mass track intensities
    (Python 3.8+ required by asari)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:3e0e65a7e74d099f79b41a7e3324e69759a79060ba82b25b7e4092700519c6f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - asari
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-cselectivity-fraction-calculation

## Summary

Implements asari's chromatographic peak selectivity (cSelectivity) quality metric on mass track (EIC) data: the fraction of data points above half the peak height that belong to the evaluated peak. It enables programmatic peak-quality assessment that reproduces the values exported with asari feature tables.

## When to use

Use when validating or re-implementing asari peak quality metrics on extracted ion chromatograms, or when assessing whether a chromatographic peak in a composite mass track is selective (cSelectivity up to 1.0) versus contaminated by co-eluting signal.

## When NOT to use

- Input is a low-resolution or very high noise LC-MS dataset — asari is mostly tested on high-resolution Orbitrap platforms and does not work with low resolution data
- You only need the exported quality values — cSelectivity and snr are already carried in full_Feature_table.tsv and can be filtered directly (snr > 2, peakshape > 0.5)
- Analysis targets MS/MS spectra — these are ignored by asari's default LC-MS workflow

## Inputs

- export/full_Feature_table.tsv from an asari project (columns include peak height, left_base/right_base boundaries, parent_masstrack_id, snr, cSelectivity, peakshape)
- export/cmap.pickle mapping parent_masstrack_id to composite mass track intensity vectors

## Outputs

- Per-feature cSelectivity values computed on the composite mass track
- Comparison table of self-computed vs asari-exported cSelectivity per feature

## How to apply

Load each feature's parent_masstrack_id, left_base/right_base boundaries, and asari-reported cSelectivity from export/full_Feature_table.tsv, then retrieve the composite mass track intensity vector from export/cmap.pickle and slice the peak region defined by the boundaries. Filter the chromatogram at half the peak height and compute cSelectivity as the fraction of data points belonging to any peak — equal to 1.0 when no noise exceeds half of any peak height — mirroring the logic of peaks.evaluate_gaussian_peak_on_intensity_list and peaks.__peaks_cSelectivity_stats_. To make the reference peak regions match those the software itself detected, reproduce the detection parameters of peaks.stats_detect_elution_peaks: prominence initialized to one-third of min_peak_height (default 1e5 for Orbitrap data), raised to at least the track noise level, plus min_timepoints-based distance and width constraints. Finally, compare self-computed cSelectivity against the exported values as in Fig. 6a-b of the paper.

## Related tools

- **asari** (Source of the cSelectivity definition and exporter of full_Feature_table.tsv and cmap.pickle used to compute and validate the metric) — https://github.com/shuzhao-li/asari
- **Python** (Implementation language for recomputing cSelectivity from mass track intensities (Python 3.8+ required by asari))

## Examples

```
import pickle; fts = read_table('export/full_Feature_table.tsv'); tracks = pickle.load(open('export/cmap.pickle','rb')); y = tracks[fts['parent_masstrack_id'][i]][fts['left_base'][i]:fts['right_base'][i]]
```

## Evaluation signals

- Self-computed cSelectivity matches the asari-exported value for the same feature within acceptable tolerance (as in Fig. 6a-b)
- A peak whose local region contains only its own high-intensity points yields cSelectivity of exactly 1.0 (e.g. the F122/F46 peaks in the documented empirical compound example)
- Peaks passing asari's quality standards report snr > 2 and peakshape > 0.5 in full_Feature_table.tsv
- Peak regions sliced from cmap.pickle align with those detected by peaks.stats_detect_elution_peaks using prominence ≥ track noise level and min_timepoints-based distance/width constraints

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings need optimization for other instruments, so thresholds like min_peak_height 1e5 may not transfer
- Data of very high noise may need customized processing, which can distort half-peak-height filtering and cSelectivity
- cSelectivity is defined on composite mass tracks, so it reflects the whole acquisition region and may not isolate sample-specific peak contamination

## Evidence

- [methods] cSelectivity definition after half-height filtering: [evidence span withheld — anchor status: not_found]
- [abstract] quality metrics exported with feature tables: [evidence span withheld — anchor status: not_found]
- [methods] peak detection parameters for reference regions: [evidence span withheld — anchor status: not_found]
- [readme] README on quality filtering in exported table: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards (part of input parameters but default values are fine for most people)."
- [readme] asari tracks peak quality metrics by design: "Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases"
