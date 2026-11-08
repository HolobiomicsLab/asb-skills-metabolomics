---
name: asari-feature-table-metric-verification
description: Use when you need to independently verify or reimplement the snr and
  cSelectivity values exported with asari feature tables, using parent_masstrack_id
  links to composite mass track (EIC) intensity vectors and left_base/right_base peak
  boundaries.
when_to_use_negative:
- Input is low-resolution MS data — the paper states 'The software does not work with
  low resolution data.'
- Data from non-Orbitrap platforms where 'The settings will need further optimization
  for other platforms' — the prominence/noise defaults (e.g. min_peak_height 1e5)
  are Orbitrap-tuned.
- The task is de novo peak detection from raw mzML rather than verification of already-exported
  asari feature-table metrics.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Produces the full_Feature_table.tsv and cmap.pickle whose snr and cSelectivity
    metrics are being reproduced; its peaks module (compute_noise_by_flanks, evaluate_gaussian_peak_on_intensity_list,
    __peaks_cSelectivity_stats_, stats_detect_elution_peaks) defines the reference
    implementations
  repo: https://github.com/shuzhao-li-lab/asari
- name: Python
  role: Implementation language for loading the TSV feature table and cmap pickle,
    slicing mass track intensity vectors, and recomputing the metrics (asari requires
    Python 3.8+)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:506a734964ab0ea019f795407c3822a05972ed6bcb57817d0404cef52223ca9d
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
n_claims: 8
---

# asari-feature-table-metric-verification

## Summary

Reproduce asari's exported peak-quality metrics — SNR (peak height over flank noise from up to 100 non-peak points per side) and cSelectivity (fraction of points above half peak height belonging to the peak) — directly from the composite mass tracks in export/cmap.pickle and the peak boundaries in full_Feature_table.tsv, to verify feature quality without manual chromatogram inspection.

## When to use

Use when you need to independently verify or reimplement the snr and cSelectivity values exported with asari feature tables, using parent_masstrack_id links to composite mass track (EIC) intensity vectors and left_base/right_base peak boundaries.

## When NOT to use

- Input is low-resolution MS data — the paper states 'The software does not work with low resolution data.'
- Data from non-Orbitrap platforms where 'The settings will need further optimization for other platforms' — the prominence/noise defaults (e.g. min_peak_height 1e5) are Orbitrap-tuned.
- The task is de novo peak detection from raw mzML rather than verification of already-exported asari feature-table metrics.

## Inputs

- export/full_Feature_table.tsv (asari-exported feature table with peak height, left_base/right_base boundaries, parent_masstrack_id, snr, cSelectivity, peakshape)
- export/cmap.pickle (composite mass track intensity vectors keyed by parent_masstrack_id)
- asari processing parameters, notably min_peak_height (default 1e5 for Orbitrap data) and min_timepoints

## Outputs

- Per-feature recomputed SNR values (peak height / mean flank noise)
- Per-feature recomputed cSelectivity values (fraction of above-half-height points belonging to the peak)
- A discrepancy report comparing recomputed vs asari-exported snr and cSelectivity for the same features

## How to apply

Load export/full_Feature_table.tsv, which carries each feature's peak height, left_base/right_base boundaries, parent_masstrack_id, and the asari-reported snr and cSelectivity values. For each feature, retrieve the composite mass track intensity vector via parent_masstrack_id from export/cmap.pickle and slice the peak region defined by left_base/right_base. Compute SNR as peak height divided by the mean intensity of up to 100 non-peak data points on each side of the peak flanks, mirroring peaks.compute_noise_by_flanks. Compute cSelectivity after filtering the chromatogram at half the peak height, as the fraction of data points belonging to the evaluated peak, following peaks.evaluate_gaussian_peak_on_intensity_list and peaks.__peaks_cSelectivity_stats_ (cSelectivity = 1.0 when no noise exceeds half of any peak height). To match the reference peak regions, reproduce the peak-detection parameters of peaks.stats_detect_elution_peaks: prominence initialized to one-third of min_peak_height (default 1e5 for Orbitrap data), raised to at least the track noise level, plus min_timepoints-based distance and width constraints. Compare self-computed values against the asari-exported values as in Fig. 6a-b of the paper.

## Related tools

- **asari** (Produces the full_Feature_table.tsv and cmap.pickle whose snr and cSelectivity metrics are being reproduced; its peaks module (compute_noise_by_flanks, evaluate_gaussian_peak_on_intensity_list, __peaks_cSelectivity_stats_, stats_detect_elution_peaks) defines the reference implementations) — https://github.com/shuzhao-li-lab/asari
- **Python** (Implementation language for loading the TSV feature table and cmap pickle, slicing mass track intensity vectors, and recomputing the metrics (asari requires Python 3.8+))

## Examples

```
import pandas as pd, pickle; ft = pd.read_csv('export/full_Feature_table.tsv', sep='\t'); cmap = pickle.load(open('export/cmap.pickle','rb')); ints = cmap[ft.parent_masstrack_id[0]]
```

## Evaluation signals

- Recomputed SNR matches the asari-exported snr for the same features within acceptable tolerance (as in Fig. 6a-b).
- Recomputed cSelectivity matches exported values, which range up to 1.0 (e.g. 1.0 for the F122/F46 peaks in the documented empirical compound example).
- Exported peaks satisfy the passing thresholds snr > 2 and peakshape > 0.5.
- Detected peak regions match those of peaks.stats_detect_elution_peaks when prominence is initialized to one-third of min_peak_height and raised to at least the track noise level.
- Flank-noise estimate uses up to 100 non-peak data points on each side of the peak, consistent with the compute_noise_by_flanks approach.

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings (including min_peak_height default 1e5) need optimization for other instruments.
- The software does not work with low-resolution data, so metric verification is only meaningful for high-resolution acquisitions.
- Pickle files such as export/cmap.pickle are for internal use — moving the result folder is fine but modifying these files is not advisable.

## Evidence

- [methods] For each m/z value, corresponding mass tracks from all sample files are summarized into one composite mass track.: "For each m/z value, corresponding mass tracks from all sample files are summarized into one composite mass track."
- [readme] All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards (part of input parameters but default values are fine for most people)."
- [readme] Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases: "Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases"
- [readme] The dashboard uses these files under the result folder: 'project.json', 'export/cmap.pickle', 'export/epd.pickle' and 'export/full_Feature_table.tsv'.: "The dashboard uses these files under the result folder: 'project.json', 'export/cmap.pickle', 'export/epd.pickle' and 'export/full_Feature_table.tsv'."
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms.: "Asari has a short history and has been mostly tested on the Orbitrap platforms."
