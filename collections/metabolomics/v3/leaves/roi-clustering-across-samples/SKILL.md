---
name: roi-clustering-across-samples
description: Use when you have already detected ROIs on each GC-IMS sample in a dataset
  and need one consolidated list of peaks, with per-sample intensity values, to feed
  peak integration and peakTable().
when_to_use_negative:
- Before per-sample peak detection has been run — there are no ROIs to cluster yet
- When samples have not been aligned in drift and retention time, so ROI coordinates
  are not comparable across samples
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCIMS
  role: Provides the ROI clustering step that merges per-sample detected ROIs into
    a single peak_list for integration and peak table building
  repo: https://github.com/sipss/GCIMS
- name: R
  role: Language in which the clustering and downstream integration/peak-table calls
    are executed
  repo: https://github.com/sipss/GCIMS
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.chemolab.2023.104938
    title: 'GCIMS: An R package for untargeted gas chromatography – Ion mobility spectrometry
      data processing'
  source_content_hash: sha256:f2df356811959775a474bec9b3bd00b0fde102210abee55deeea51a3c90951b3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/sipss/GCIMS
  repo_url: https://github.com/sipss/GCIMS
  tools:
  - BiocParallel
  - GCIMS
  - R
  - cowplot
  - dplyr
  - ggplot2
  - readr
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ROI clustering across samples

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups regions of interest (ROIs) detected independently in each GC-IMS sample into a shared, sample-consistent peak list so that peaks can be integrated and tabulated across samples. It is the step that turns per-sample peak detections into a single cross-sample peak inventory in the GCIMS R package.

## When to use

Use when you have already detected ROIs on each GC-IMS sample in a dataset and need one consolidated list of peaks, with per-sample intensity values, to feed peak integration and peakTable().

## When NOT to use

- Before per-sample peak detection has been run — there are no ROIs to cluster yet
- When samples have not been aligned in drift and retention time, so ROI coordinates are not comparable across samples

## Inputs

- GCIMSDataset with per-sample detected ROIs (output of peak detection on each sample)
- Dataset whose samples have been aligned in drift time and retention time

## Outputs

- peak_clustering result containing a unified peak_list spanning all samples
- Clustered peak list usable as input to integratePeaks() and, after integration, peakTable()

## How to apply

After running peak detection on each preprocessed GC-IMS sample (with noise_level first optimized on a single sample, e.g. by filtering rt = c(0, 1100) s and dt = c(5, 16) ms and smoothing with a Savitzky-Golay filter), collect the per-sample ROI lists and cluster ROIs across samples using the GCIMS package's ROI clustering step. The clustering decides which detections in different samples correspond to the same chemical peak, producing a single peak_list object from the peak_clustering result. This list is what downstream steps expect: integratePeaks() is called with peak_clustering$peak_list, and peakTable() consumes the integrated peaks (with aggregate_conflicting_peaks = max for conflicting values). Alignment of samples in drift and retention time should precede detection so that ROI coordinates are comparable across samples; the baseline estimated with dt_peak_fwhm_ms = 0.2, dt_region_multiplier = 12, rt_length_s = 200 is subtracted before detection. Missing peaks for particular samples will surface as NA entries in the resulting peak table, which imputePeakTable() can fill afterward.

## Related tools

- **GCIMS** (Provides the ROI clustering step that merges per-sample detected ROIs into a single peak_list for integration and peak table building) — https://github.com/sipss/GCIMS
- **R** (Language in which the clustering and downstream integration/peak-table calls are executed) — https://github.com/sipss/GCIMS

## Examples

```
dataset <- integratePeaks(dataset, peak_clustering$peak_list, ...)
```

## Evaluation signals

- The peak_list returned by clustering contains ROIs drawn from every sample in the dataset, not just a single sample
- peakTable() on the integrated clustered peaks yields a peak_table_matrix with one row per clustered peak across samples
- Each row of the peak table has an intensity entry for every sample; samples lacking a given peak appear as NA rather than silently dropping the row
- No errors are thrown when peak_clustering$peak_list is passed to integratePeaks() and peakTable() on the threeketones demo dataset

## Limitations

- Clustered peaks missing from some samples produce NA values in the peak table, requiring imputePeakTable() for downstream analysis
- Quality of clustering depends on prior sample alignment in drift time and retention time; unaligned samples can fragment or merge peaks incorrectly
- Conflicting peak values for clustered ROIs must be resolved at peakTable() time, e.g. with aggregate_conflicting_peaks = max

## Evidence

- [intro] ROI clustering produces a unified peak list consumed by integratePeaks and peakTable: "clustering of ROIs across samples"
- [intro] Per-sample peak detection precedes clustering; noise_level is tuned on one sample: "First try one sample and optimize the `noise_level` parameter there."
- [intro] Clustering result's peak_list is the direct input to peak integration: "dataset <- integratePeaks(
  dataset, 
  peak_clustering$peak_list, "
- [intro] Sample alignment is required before ROI coordinates are comparable across samples: "In order to be able to compare samples to each other, we align the samples."
- [intro] Missing peaks after clustering appear as NA and need imputation: "peak_table_imputed <- imputePeakTable(peak_table$peak_table_matrix,
                                      dataset, "
- [readme] GCIMS package implements the clustering within a full preprocessing pipeline: "GCIMS is an R package implementing a preprocessing pipeline for Gas Chromatography – Ion
Mobility Spectrometry samples."
