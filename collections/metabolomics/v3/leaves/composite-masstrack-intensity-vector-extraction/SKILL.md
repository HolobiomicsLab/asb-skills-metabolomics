---
name: composite-masstrack-intensity-vector-extraction
description: Use when a full_Feature_table.tsv feature needs its underlying extracted-ion-chromatogram
  trace — e.g. to verify peak shape, recompute SNR/cSelectivity, or backtrack a feature
  to its mass track.
when_to_use_negative:
- Input is raw vendor Thermo .RAW or profile mzML files — convert to centroided mzML
  first (ThermoRawFileParser/msConvert) and run asari processing before mass tracks
  exist
- Input is already an annotated compound table (e.g. unique_compound__Feature_table.tsv)
  and only feature-level values are needed, not chromatographic traces
- Data is low-resolution or from very long chromatography — the article notes asari
  does not work with low-resolution data and its efficiency is not optimal on longer
  chromatography
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Produces the cmap.pickle and full_Feature_table.tsv exports that this extraction
    consumes; mass track IDs link features to composite EIC vectors
  repo: https://github.com/shuzhao-li/asari
- name: Python
  role: Runtime for unpickling cmap.pickle, slicing intensity vectors by left_base/right_base,
    and recomputing metrics
- name: asari dashboard
  role: Alternative interactive route to visualize the same cmap.pickle mass tracks
    per feature
  repo: https://panel.holoviz.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:81826410bf5baac4d7d4714bb79ea78207baad429823343d6e01c5264963f31b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - asari
  - asari dashboard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# composite-masstrack-intensity-vector-extraction

## Summary

Retrieves the composite mass track (EIC) intensity vector underlying an asari feature by looking up its parent_masstrack_id in export/cmap.pickle, enabling peak-quality inspection, SNR/cSelectivity recomputation, and feature-to-EIC backtracking.

## When to use

Use when a full_Feature_table.tsv feature needs its underlying extracted-ion-chromatogram trace — e.g. to verify peak shape, recompute SNR/cSelectivity, or backtrack a feature to its mass track.

## When NOT to use

- Input is raw vendor Thermo .RAW or profile mzML files — convert to centroided mzML first (ThermoRawFileParser/msConvert) and run asari processing before mass tracks exist
- Input is already an annotated compound table (e.g. unique_compound__Feature_table.tsv) and only feature-level values are needed, not chromatographic traces
- Data is low-resolution or from very long chromatography — the article notes asari does not work with low-resolution data and its efficiency is not optimal on longer chromatography

## Inputs

- asari project export directory containing export/cmap.pickle
- export/full_Feature_table.tsv with columns parent_masstrack_id, left_base, right_base, peak_height, snr, cSelectivity
- project.json for processing parameters and track metadata

## Outputs

- composite mass track intensity vector (EIC) for the feature's parent mass track
- peak-region intensity slice bounded by left_base and right_base
- recomputed quality metrics (SNR, cSelectivity) comparable to exported feature-table values

## How to apply

Load the asari project's export/cmap.pickle, a dict of composite mass tracks keyed by mass track ID. Read the feature row from export/full_Feature_table.tsv (or preferred_Feature_table.tsv), take its parent_masstrack_id, and fetch the corresponding intensity vector. Composite mass tracks exist because asari first extracts mass tracks of unique m/z values from each mzML file, then summarizes the corresponding mass tracks from all sample files into one composite track per m/z — so one lookup yields the full cross-sample chromatogram rather than per-sample EICs. Slice the vector using the feature's left_base and right_base columns to isolate the peak region, and use the peak_height column (with default min_peak_height 1e5 for Orbitrap data) when deriving metrics such as SNR (peak height over mean flank noise, up to 100 non-peak points per side) or cSelectivity (fraction of points above half peak height belonging to the peak).

## Related tools

- **asari** (Produces the cmap.pickle and full_Feature_table.tsv exports that this extraction consumes; mass track IDs link features to composite EIC vectors) — https://github.com/shuzhao-li/asari
- **Python** (Runtime for unpickling cmap.pickle, slicing intensity vectors by left_base/right_base, and recomputing metrics)
- **asari dashboard** (Alternative interactive route to visualize the same cmap.pickle mass tracks per feature) — https://panel.holoviz.org/

## Examples

```
import pickle; cmaps = pickle.load(open('export/cmap.pickle','rb')); vec = cmaps[row['parent_masstrack_id']]; peak = vec[row['left_base']:row['right_base']+1]
```

## Evaluation signals

- The parent_masstrack_id from the feature row resolves to a key in the unpickled cmap.pickle dict without KeyError
- The sliced peak region contains a maximum whose intensity matches the feature-table peak_height within tolerance
- Recomputed SNR and cSelectivity reproduce the asari-exported snr and cSelectivity values within acceptable tolerance (e.g. Fig. 6a-b); exported cSelectivity ranges up to 1.0
- Peak regions correspond to detected peaks: prominence was initialized to one-third of min_peak_height (default 1e5 for Orbitrap), raised to at least track noise, with min_timepoints distance/width constraints
- Features retained in full_Feature_table.tsv satisfy snr > 2 and peakshape > 0.5

## Limitations

- Pickle files are for internal use — the README warns that modification of export/cmap.pickle etc. is not a good idea and pickle files are removed/kept only for internal processing
- Settings were mostly tested on Orbitrap platforms and will need optimization for other instruments; the tool does not work with low-resolution data
- MS/MS spectra are ignored by asari, so composite mass tracks carry MS1 intensity only

## Evidence

- [methods] From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid.: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
- [methods] composite track summarization: "For each m/z value, corresponding mass tracks from all sample files are summarized into one composite mass track."
- [readme] dashboard file dependencies: "The dashboard uses these files under the result folder: 'project.json', 'export/cmap.pickle', 'export/epd.pickle' and 'export/full_Feature_table.tsv'."
- [readme] tracking between features and EICs: "Reproducible, track and backtrack between features and mass tracks (EICs)"
- [readme] peak detection on composite map: "Peak detection on a composite map instead of repeated on individual samples"
