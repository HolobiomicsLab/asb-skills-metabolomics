---
name: extracted-ion-chromatogram-isolation-within-tic-peaks
description: Use when raw scan-level metabolomics data (mzML, mzXML, CDF) plus detected
  TIC peak boundaries are available and centWave parameter estimation—especially the
  ppm mass-error parameter—requires isolating EICs within each TIC peak's retention-time
  window.
when_to_use_negative:
- The input is already a processed feature table rather than raw scan-level data,
  since parameter inference must occur on raw data prior to feature-table generation
- The goal is retention-time correction across samples, because the divide-and-conquer
  approach does not compare EIC peaks across samples
- The target parameter is mzDiff, the one continuous centWave parameter not optimized
  by AutoTuner (only a sensitivity analysis is supported)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: R package that isolates EICs within detected TIC peaks and infers the seven
    continuous centWave parameters (including ppm) from them in a single statistical-inference
    step
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Implementation environment; AutoTuner is distributed as an R package via BioConductor
    and requires R >= 3.6
- name: XCMS
  role: Consumer of the estimated centWave peak-picking parameters (S/N, scan count,
    noise, prefilter, peakwidth, ppm) for downstream feature detection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:8c6bb5eb3f3f3b072118b2a52df2dcc5c5bda2995d43b7e16b0a7060535617a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# extracted-ion-chromatogram-isolation-within-tic-peaks

## Summary

Within each detected TIC peak of raw untargeted metabolomics data, isolate the extracted-ion chromatograms (EICs) of the ions eluting in that peak's retention-time window so that scan-level properties (m/z deviations, S/N, noise, prefilter, peakwidth) can be used to estimate centWave parameters, most notably the mass error in ppm.

## When to use

Use when raw scan-level metabolomics data (mzML, mzXML, CDF) plus detected TIC peak boundaries are available and centWave parameter estimation—especially the ppm mass-error parameter—requires isolating EICs within each TIC peak's retention-time window.

## When NOT to use

- The input is already a processed feature table rather than raw scan-level data, since parameter inference must occur on raw data prior to feature-table generation
- The goal is retention-time correction across samples, because the divide-and-conquer approach does not compare EIC peaks across samples
- The target parameter is mzDiff, the one continuous centWave parameter not optimized by AutoTuner (only a sensitivity analysis is supported)

## Inputs

- Raw scan-level mass-spectral data (m/z, intensity, retention time per scan) from at least 3 samples in .mzML, .mzXML, or .CDF format
- Detected TIC peak boundaries (retention-time windows) from AutoTuner's sliding window analysis
- Spreadsheet matching raw data sample names to experimental factor columns

## Outputs

- Isolated EICs per detected TIC peak
- Ppm-per-bin values filtered by scan-continuity, with noise bins removed
- Gaussian-KDE empirical distribution of ppm values (KLD<0.5 subsampling)
- Estimated ppm parameter (sd of clustered ppm values above 1) and scan-level estimates of S/N threshold, scan count, noise, prefilter intensity, and peakwidth range, compiled into a structured seven-parameter record

## How to apply

Load the raw scan-level mass-spectral data (m/z, intensity, retention time per scan) together with the detected TIC peak boundaries from the sliding-window peak detection step. For each detected TIC peak, isolate the EICs of the ions eluting within that peak's retention-time window. Within each TIC peak's EICs, sort and bin the m/z values and filter out bins failing the scan-continuity criterion as noise; compute ppm per surviving bin, build a Gaussian-KDE empirical distribution of ppm values (subsampled with KLD<0.5 checks), then apply k-means clustering with an outlier score to keep ppm values above 1, returning sd(x) of those values as the ppm estimate. Use the same isolated EICs to estimate the remaining centWave parameters (S/N threshold, scan count, noise, prefilter intensity, and peakwidth range) from their scan-level properties. The rationale is a 'divide-and-conquer' single-step statistical inference on raw data instead of iterative parameter sweeps, which reduces the centWave parameter combinations from at least 24*5^8 possible choices to 40.

## Related tools

- **AutoTuner** (R package that isolates EICs within detected TIC peaks and infers the seven continuous centWave parameters (including ppm) from them in a single statistical-inference step) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Implementation environment; AutoTuner is distributed as an R package via BioConductor and requires R >= 3.6)
- **XCMS** (Consumer of the estimated centWave peak-picking parameters (S/N, scan count, noise, prefilter, peakwidth, ppm) for downstream feature detection)

## Examples

```
BiocManager::install("Autotuner")
```

## Evaluation signals

- Bins failing the scan-continuity criterion are removed as noise before ppm computation, so every ppm value in the KDE input comes from a scan-continuous bin
- Subsampling of the KDE must satisfy a KLD < 0.5 check before k-means clustering is applied
- Only ppm values above 1 are retained by the k-means/outlier-score heuristic, and the final ppm estimate equals sd(x) of those retained values
- The output record contains the seven continuous centWave parameter estimates (ppm, S/N threshold, scan count, noise, prefilter intensity, peakwidth) with peakwidth expressed as a range
- Parameter estimates should be reproducible across random sample subsets (Monte Carlo robustness), with CV < 0.1 for most parameters when ~9 samples are used

## Limitations

- The Group difference parameter is estimated through a non-automatable cross sample comparison during TIC peak detection, so its robustness could not be checked
- AutoTuner is not parallelized; parallelization is listed as a possible future improvement
- The sliding window analysis requires user input during the first portion of AutoTuner before TIC peak boundaries are available
- The divide-and-conquer speed gain comes at the loss of comparing EIC peaks across samples for retention-time correction

## Evidence

- [full_text] EIC isolation step within TIC peaks: [evidence span withheld — anchor status: not_found]
- [full_text] Ppm estimation method from EIC bins: [evidence span withheld — anchor status: not_found]
- [readme] Raw input formats required: [evidence span withheld — anchor status: external_doc]
- [intro] Single-step inference rationale: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [discussion] Parameter combination reduction: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40."
