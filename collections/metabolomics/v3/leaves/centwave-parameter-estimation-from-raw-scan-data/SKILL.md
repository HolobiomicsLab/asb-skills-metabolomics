---
name: centwave-parameter-estimation-from-raw-scan-data
description: Use when raw untargeted metabolomics data (mzML, mzXML, or CDF from qTOF,
  orbitrap, or FT-ICR analyzers) must be processed with XCMS/MZmine2 centWave and
  dataset-specific peak-picking parameters must be estimated in one pass rather than
  by iteratively testing parameter combinations.
when_to_use_negative:
- Input is already a processed feature table — AutoTuner performs statistical inference
  on raw scan-level data, not on feature tables
- You need retention-time correction parameterization across samples — AutoTuner's
  divide-and-conquer approach does not compare EIC peaks across samples to estimate
  retention time correction
- You need the Group difference parameter — it requires a non-automatable cross-sample
  comparison during TIC peak detection and cannot be estimated by this stage
tools:
- name: AutoTuner
  role: 'Implements the EIC-based parameter estimation: isolates EICs within each
    detected TIC peak, bins m/z values, fits the Gaussian KDE ppm distribution, and
    returns the seven centWave parameter estimates'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Implementation language; AutoTuner is distributed as an R package through
    BioConductor and requires R version 3.6 or greater
- name: XCMS
  role: Downstream consumer — the estimated continuous centWave parameters tune the
    centWave peak-picking algorithm used by XCMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:02d9918b87e4081432f189d155d8afb8383a002b6ae1ae9ef27a1f85066fe653
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# centwave-parameter-estimation-from-raw-scan-data

## Summary

AutoTuner's single-step statistical-inference stage that estimates the ppm mass-error parameter and the remaining continuous centWave peak-picking parameters (S/N threshold, scan count, noise, prefilter intensity, peakwidth) directly from scan-level raw data, replacing iterative parameter sweeping.

## When to use

Use when raw untargeted metabolomics data (mzML, mzXML, or CDF from qTOF, orbitrap, or FT-ICR analyzers) must be processed with XCMS/MZmine2 centWave and dataset-specific peak-picking parameters must be estimated in one pass rather than by iteratively testing parameter combinations.

## When NOT to use

- Input is already a processed feature table — AutoTuner performs statistical inference on raw scan-level data, not on feature tables
- You need retention-time correction parameterization across samples — AutoTuner's divide-and-conquer approach does not compare EIC peaks across samples to estimate retention time correction
- You need the Group difference parameter — it requires a non-automatable cross-sample comparison during TIC peak detection and cannot be estimated by this stage

## Inputs

- Raw scan-level mass-spectral data (m/z, intensity, retention time per scan) converted from proprietary instrument formats to .mzML, .mzXML, or .CDF — at least 3 samples required
- Detected TIC peak boundaries (retention-time windows) from AutoTuner's sliding-window/peak-detection stage
- Sample metadata spreadsheet with one column matching raw sample names and one describing experimental factors

## Outputs

- ppm (mass error) parameter estimate: sd(x) of the k-means-retained, KDE-modeled ppm values above 1
- Estimates for the remaining continuous centWave parameters: S/N threshold, Scan count, Noise, Prefilter intensity, and Peak-width range
- A structured record of all seven continuous centWave parameter estimates written as the output file

## How to apply

Load the raw scan-level mass-spectral data (m/z, intensity, retention time per scan) together with the detected TIC peak boundaries; for each TIC peak, isolate the extracted-ion chromatograms (EICs) of ions eluting within that peak's retention-time window. For the ppm estimate: sort and bin m/z values within the EICs, filter out bins failing the scan-continuity criterion as noise, compute ppm per bin, fit a Gaussian kernel-density empirical distribution to the deviations (subsampling with KLD checks below 0.5), then apply k-means clustering with an outlier score and keep ppm values above 1, returning sd(x) of the retained values as the ppm estimate. Estimate the remaining centWave parameters — S/N threshold, Scan count, Noise, Prefilter intensity, and Peak-width range — from the scan-level properties of the isolated EICs. Compile the seven continuous parameter estimates into a structured record and write it as the output file; mzDiff is deliberately not optimized (sensitivity analysis showed distinct values had minimal effect on the returned feature table).

## Related tools

- **AutoTuner** (Implements the EIC-based parameter estimation: isolates EICs within each detected TIC peak, bins m/z values, fits the Gaussian KDE ppm distribution, and returns the seven centWave parameter estimates) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Implementation language; AutoTuner is distributed as an R package through BioConductor and requires R version 3.6 or greater)
- **XCMS** (Downstream consumer — the estimated continuous centWave parameters tune the centWave peak-picking algorithm used by XCMS)

## Examples

```
BiocManager::install("Autotuner")
```

## Evaluation signals

- Retained ppm values are all above 1 (the k-means/outlier-score heuristic discards values at or below 1)
- Gaussian-KDE subsampling terminates with KLD checks below the 0.5 threshold
- The returned ppm estimate equals sd(x) of the values in the retained k-means cluster
- Monte Carlo robustness: parameter estimates from random subsets of samples should be similar; 9–12 samples yield CV values less than 0.1 for most parameters
- The output record contains all seven continuous centWave parameter estimates (ppm plus S/N threshold, scan count, noise, prefilter intensity, peakwidth)

## Limitations

- AutoTuner is not parallelized; parallelization is listed as a possible future improvement
- The divide-and-conquer speed gain comes at the loss of comparing EIC peaks across samples to estimate retention time correction
- The sliding window (TIC peak detection) stage requires user input; robustness of the Group difference estimate could not be checked because it is estimated through a non-automatable cross-sample comparison
- mzDiff is the one continuous centWave parameter not optimized — only a sensitivity analysis was performed

## Evidence

- [intro] The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values.: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [intro] AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [discussion] the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters: "the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters"
- [readme] AutoTuner is used to identify dataset specific parameters to process untargeted metabolomics data.: [evidence span withheld — anchor status: external_doc]
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
