---
name: drift-time-retention-time-sample-alignment
description: Use when you have multiple GC-IMS spectra loaded into a GCIMSDataset
  (e.g. threeketones or urine samples) and need them comparable across samples before
  peak detection and ROI clustering.
when_to_use_negative:
- You only have a single GC-IMS sample and no cross-sample comparison is planned (alignment
  corrects inter-sample shifts and is unnecessary).
- Your data are not GC-IMS spectra (e.g. LC-MS or GC-MS runs) — the GCIMS package
  alignment operates on drift time in ms and retention time in s specific to ion mobility
  spectrometry.
- You have already built the peak table; alignment must precede peak detection and
  ROI clustering, not follow peakTable().
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GCIMS
  role: Supplies the alignment routines that correct drift-time and retention-time
    shifts between samples within a GCIMSDataset.
  repo: https://github.com/sipss/GCIMS
- name: R
  role: Runtime environment in which the GCIMSDataset alignment is executed.
- name: cowplot
  role: Used to grid aligned sample spectra (plot_grid) for visual verification that
    peaks co-localize after alignment.
- name: ggplot2
  role: Rendering backend for plotting the aligned GC-IMS samples during visual QC.
- name: BiocParallel
  role: Parallel backend for running alignment and downstream GCIMS pipeline steps
    over multiple samples.
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.chemolab.2023.104938
    title: 'GCIMS: An R package for untargeted gas chromatography – Ion mobility spectrometry
      data processing'
  source_content_hash: sha256:9f079166036a11f5950790455c2389e6df2d1ffa75a5675c1c6aecc789af0aa0
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# drift-time-retention-time-sample-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align GC-IMS samples to each other in both drift time (ms) and retention time (s) so that peak detection and ROI clustering can be compared across samples using the GCIMS R package.

## When to use

Use when you have multiple GC-IMS spectra loaded into a GCIMSDataset (e.g. threeketones or urine samples) and need them comparable across samples before peak detection and ROI clustering.

## When NOT to use

- You only have a single GC-IMS sample and no cross-sample comparison is planned (alignment corrects inter-sample shifts and is unnecessary).
- Your data are not GC-IMS spectra (e.g. LC-MS or GC-MS runs) — the GCIMS package alignment operates on drift time in ms and retention time in s specific to ion mobility spectrometry.
- You have already built the peak table; alignment must precede peak detection and ROI clustering, not follow peakTable().

## Inputs

- GCIMSDataset object holding multiple smoothed and optionally decimated GC-IMS intensity matrices (drift time in ms, retention time in s)
- GC-IMS sample files, e.g. the 'threeketones' demo dataset or urine volatile-compounds samples

## Outputs

- Aligned GCIMSDataset in which homologous peaks share consistent drift-time (ms) and retention-time (s) coordinates across samples
- Overlaid/aligned spectra suitable for subsequent cross-sample peak detection and ROI clustering

## How to apply

After loading samples into a GCIMSDataset and filtering to the region of interest (e.g. filterRt(rt = c(0, 1100)) and filterDt(dt = c(5, 16)) on the threeketones demo), apply the GCIMS alignment routines to correct retention-time and drift-time shifts between samples so their intensity matrices can be directly compared. Alignment is performed on the smoothed/decimated spectra (Savitzky-Golay in both dimensions, optional decimation) since noise and memory load affect the warp estimates. Align in drift time and in retention time as two separate correction steps, then optionally correct the baseline (dt_peak_fwhm_ms = 0.2, dt_region_multiplier = 12, rt_length_s = 200) before peak detection. Verify alignment by plotting overlaid samples (e.g. with cowplot::plot_grid of plot(ds$getSample(...))) and checking that homologous peaks occupy the same drift-time/retention-time grid before clustering ROIs across samples.

## Related tools

- **GCIMS** (Supplies the alignment routines that correct drift-time and retention-time shifts between samples within a GCIMSDataset.) — https://github.com/sipss/GCIMS
- **R** (Runtime environment in which the GCIMSDataset alignment is executed.)
- **cowplot** (Used to grid aligned sample spectra (plot_grid) for visual verification that peaks co-localize after alignment.)
- **ggplot2** (Rendering backend for plotting the aligned GC-IMS samples during visual QC.)
- **BiocParallel** (Parallel backend for running alignment and downstream GCIMS pipeline steps over multiple samples.)

## Examples

```
library(GCIMS); ds <- filterRt(filterDt(ds, dt = c(5, 16)), rt = c(0, 1100)); ds <- alignSamples(ds)
```

## Evaluation signals

- After alignment, overlaid spectra of multiple samples show homologous peaks at consistent drift-time (ms) and retention-time (s) positions.
- Aligned samples can be clustered: ROI clustering across samples yields a coherent peak_list with one row per clustered peak.
- Alignment is applied after filtering to the region of interest (rt = c(0, 1100) s, dt = c(5, 16) ms for the threeketones demo) and after Savitzky-Golay smoothing, not before.
- The aligned GCIMSDataset retains the same number of samples and their annotations (e.g. Sample1, Sample2) as before alignment.
- Downstream peakTable(aggregate_conflicting_peaks = max) succeeds on the aligned, detected, and integrated peak list without errors.

## Limitations

- Alignment is performed in drift time and retention time as separate correction steps; residual shifts in both dimensions may persist if warping parameters are not tuned per dataset.
- Decimation and smoothing (Savitzky-Golay) affect the alignment input; overly aggressive decimation may degrade warping accuracy.
- Vendor-dependent GC-IMS data formats must be imported into the GCIMS package before alignment, which is handled separately in a dedicated vignette.
- Delaying modification of samples in a batch (delayed operations) is supported, but alignment must still be realized before peak detection runs on the aligned matrices.

## Evidence

- [intro] In order to be able to compare samples to each other, we align the samples.: "In order to be able to compare samples to each other, we align the samples."
- [intro] alignment of samples in drift time and retention time: [evidence span withheld — anchor status: not_found]
- [intro] dataset <- estimateBaseline(
  dataset,
  dt_peak_fwhm_ms = 0.2, 
  dt_region_multiplier = 12,
  rt_length_s = 200
): "dataset <- estimateBaseline(
  dataset,
  dt_peak_fwhm_ms = 0.2, 
  dt_region_multiplier = 12,
  rt_length_s = 200
)"
- [intro] cowplot::plot_grid(
  plot(ds$getSample("Sample1")),
  plot(ds$getSample("Sample2")),
  ncol = 2
): "cowplot::plot_grid(
  plot(ds$getSample("Sample1")),
  plot(ds$getSample("Sample2")),
  ncol = 2
)"
- [readme] GCIMS is an R package implementing a preprocessing pipeline for Gas Chromatography – Ion Mobility Spectrometry samples.: "GCIMS is an R package implementing a preprocessing pipeline for Gas Chromatography – Ion Mobility Spectrometry samples."
