---
name: chromatographic-snr-computation-from-eic-flanks
description: 'Use when validating asari feature quality without manual inspection:
  you have full_Feature_table.tsv plus export/cmap.pickle and must recompute snr,
  cSelectivity, or peakshape to check exported values.'
when_to_use_negative:
- Input is already an asari-preferred_Feature_table.tsv where exported quality metrics
  can be used directly and no metric reimplementation is needed
- Data is low-resolution or from non-Orbitrap platforms without re-optimized parameters
  — the article states asari does not work with low resolution data and settings are
  mostly tested on Orbitrap platforms
- The goal is peak detection itself rather than post-hoc quality assessment of already-detected
  features
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Source of the mass tracks, feature table columns (left_base/right_base, parent_masstrack_id,
    snr, cSelectivity) and the reference functions compute_noise_by_flanks and __peaks_cSelectivity_stats_
    whose behavior is being mirrored
  repo: https://github.com/shuzhao-li-lab/asari
- name: Python
  role: Implementation language for loading cmap.pickle, slicing peak regions from
    EIC vectors, and computing the SNR and cSelectivity metrics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:7f757b6697f3ceb7263a42b2f4d5e654d4466f28fc36f0ff47b3b95042fe007a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - Python
  - asari
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Chromatographic SNR computation from EIC flanks

## Summary

Reproduces asari's peak quality metrics — SNR (peak height divided by flank noise) and cSelectivity (fraction of high-intensity points after half-peak-height filtering that belong to the evaluated peak) — directly on mass track (EIC) intensity vectors.

## When to use

Use when validating asari feature quality without manual inspection: you have full_Feature_table.tsv plus export/cmap.pickle and must recompute snr, cSelectivity, or peakshape to check exported values.

## When NOT to use

- Input is already an asari-preferred_Feature_table.tsv where exported quality metrics can be used directly and no metric reimplementation is needed
- Data is low-resolution or from non-Orbitrap platforms without re-optimized parameters — the article states asari does not work with low resolution data and settings are mostly tested on Orbitrap platforms
- The goal is peak detection itself rather than post-hoc quality assessment of already-detected features

## Inputs

- asari export/full_Feature_table.tsv with peak height, left_base/right_base boundaries, parent_masstrack_id, and exported snr/cSelectivity columns
- asari export/cmap.pickle mapping parent_masstrack_id to composite mass track (EIC) intensity vectors
- asari processing parameters, notably min_peak_height (default 1e5 for Orbitrap data) and min_timepoints

## Outputs

- Per-feature recomputed SNR values (peak height / mean flank noise of up to 100 non-peak points per side)
- Per-feature recomputed cSelectivity values in [0, 1] after half-peak-height filtering
- A discrepancy comparison against the asari-exported snr and cSelectivity columns (as in Fig. 6a-b)

## How to apply

For each feature in full_Feature_table.tsv, look up its composite mass track via parent_masstrack_id in export/cmap.pickle and slice the peak region using left_base/right_base. Compute SNR as peak height divided by the mean intensity of non-peak data points, sampling up to 100 such points on each side of the peak flanks, mirroring peaks.compute_noise_by_flanks. Compute cSelectivity per the logic of peaks.evaluate_gaussian_peak_on_intensity_list and peaks.__peaks_cSelectivity_stats_: filter the chromatogram at half the peak height and take the fraction of the remaining high-intensity points that belong to the evaluated peak (1.0 when no noise exceeds half of any peak height). To match the software's own peak regions, reproduce the parameters of peaks.stats_detect_elution_peaks: prominence initialized to one-third of min_peak_height (default 1e5 for Orbitrap data), raised to at least the track noise level, plus min_timepoints-based distance and width constraints. Finally, compare self-computed values against the asari-exported snr and cSelectivity columns; published peaks report snr > 2 and peakshape > 0.5, with cSelectivity up to 1.0.

## Related tools

- **asari** (Source of the mass tracks, feature table columns (left_base/right_base, parent_masstrack_id, snr, cSelectivity) and the reference functions compute_noise_by_flanks and __peaks_cSelectivity_stats_ whose behavior is being mirrored) — https://github.com/shuzhao-li-lab/asari
- **Python** (Implementation language for loading cmap.pickle, slicing peak regions from EIC vectors, and computing the SNR and cSelectivity metrics)

## Examples

```
import pickle; tracks = pickle.load(open('export/cmap.pickle','rb')); vec = tracks[parent_masstrack_id]; peak = vec[left_base:right_base+1]
```

## Evaluation signals

- Recomputed SNR matches the asari-exported snr value for the same feature within acceptable tolerance
- Recomputed cSelectivity matches the exported cSelectivity; values range up to 1.0 (e.g. 1.0 for the F122/F46 peaks in the documented empirical compound example)
- Exported peaks in the feature table pass the thresholds snr > 2 and peakshape > 0.5
- Detected reference peak regions match peaks.stats_detect_elution_peaks when prominence is set to max(min_peak_height/3, track noise level) with min_timepoints-based distance/width constraints
- cSelectivity equals 1.0 exactly when no non-peak point exceeds half of the peak height

## Limitations

- Default settings (min_peak_height 1e5, snr and peakshape defaults) are tuned for Orbitrap data and may not transfer to other platforms
- Asari does not work with low resolution data, so flank-noise SNR is undefined/unreliable there
- Data of very high noise may need customized processing; borderline cases trade sensitivity against true discovery rate

## Evidence

- [other] Asari computes cSelectivity after filtering the chromatogram at half the peak height, as the fraction of data points belonging to any peak: [evidence span withheld — anchor status: not_found]
- [other] SNR is defined as peak height divided by the noise level, where noise is the average intensity of up to 100 non-peak data points on each side of the peak.: [evidence span withheld — anchor status: not_found]
- [methods] [peaks.stats_detect_elution_peaks](peaks.stats_detect_elution_peaks): "[peaks.stats_detect_elution_peaks](peaks.stats_detect_elution_peaks)"
- [readme] All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
- [other] These metrics, together with peak shape, are exported with the feature tables so users can assess peak quality without manual inspection.: [evidence span withheld — anchor status: not_found]
