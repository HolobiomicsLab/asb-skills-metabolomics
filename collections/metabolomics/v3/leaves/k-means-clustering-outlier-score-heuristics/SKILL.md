---
name: k-means-clustering-outlier-score-heuristics
description: Use when estimating the mass error (ppm) centWave parameter from extracted-ion
  chromatograms of detected TIC peaks in raw untargeted metabolomics data (mzML, mzXML,
  CDF), needing to separate true-feature ppm values from noise-derived ppm values.
when_to_use_negative:
- The input is already a processed feature table rather than raw scan-level m/z/intensity/retention-time
  data — AutoTuner's inference runs on raw data, not feature tables
- The goal is to estimate parameters other than ppm (e.g. S/N, Scan count, Noise,
  Prefilter, peakwidth, or the non-automatable Group difference parameter), since
  this heuristic is specific to the ppm estimation stage
- The sample set has fewer than 3 raw data samples, which AutoTuner requires as minimum
  input
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AutoTuner
  role: 'R package implementing the entire pipeline: TIC peak detection, EIC isolation,
    m/z binning with scan-continuity filtering, Gaussian-KDE fitting with KLD subsampling,
    and k-means clustering with outlier scoring to estimate the ppm parameter'
  repo: KujawinskiLaboratory/Autotuner
- name: XCMS
  role: Downstream consumer — the ppm estimate feeds the centWave peak-picking algorithm
    that XCMS (and MZmine2) uses; the skill replaces manual tuning of one of centWave's
    11 parameters
- name: R
  role: Implementation language; AutoTuner is distributed as an R package via BioConductor,
    so the k-means/KDE heuristic is invoked from R code
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:ff714694bfcfb4dee153032b9c76e08c26ed58a27663f5d40963507f1ed256b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KujawinskiLaboratory/Autotuner
  tools:
  - AutoTuner
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# k-means-clustering-outlier-score-heuristics

## Summary

A statistical-inference heuristic from AutoTuner that fits a Gaussian-KDE empirical distribution to ppm deviations from EIC bins, subsamples it via KLD<0.5 checks, then applies k-means clustering with an outlier score to retain true-feature ppm values above 1 and return their standard deviation as the ppm estimate.

## When to use

Use when estimating the mass error (ppm) centWave parameter from extracted-ion chromatograms of detected TIC peaks in raw untargeted metabolomics data (mzML, mzXML, CDF), needing to separate true-feature ppm values from noise-derived ppm values.

## When NOT to use

- The input is already a processed feature table rather than raw scan-level m/z/intensity/retention-time data — AutoTuner's inference runs on raw data, not feature tables
- The goal is to estimate parameters other than ppm (e.g. S/N, Scan count, Noise, Prefilter, peakwidth, or the non-automatable Group difference parameter), since this heuristic is specific to the ppm estimation stage
- The sample set has fewer than 3 raw data samples, which AutoTuner requires as minimum input

## Inputs

- Scan-level raw mass-spectral data (per-scan m/z, intensity, retention time) from at least 3 samples in mzML, mzXML, or CDF format
- Detected TIC peak boundaries (retention-time windows) from the AutoTuner TIC peak detection step
- Extracted-ion chromatograms (EICs) of ions eluting within each detected TIC peak

## Outputs

- A continuous ppm (mass error) estimate — the standard deviation of retained true-feature ppm values — as one of AutoTuner's seven centWave parameter estimates
- Retained set of true-feature ppm values above the outlier-score threshold, separated from noise-derived ppm values

## How to apply

Within each detected TIC peak, isolate the EICs eluting in that retention-time window, sort and bin their m/z values, and discard bins failing the scan-continuity criterion as noise. Compute ppm per surviving bin, build a Gaussian kernel-density empirical distribution over the ppm values, and use KLD<0.5 subsampling to shrink the data to a tractable size. Then run k-means clustering on the distribution and apply an outlier score to keep only ppm values above 1; return the standard deviation (sd) of the retained values as the ppm estimate. This turns a manual, iterative tuning problem into a single-step statistical inference on the raw scan-level m/z deviations.

## Related tools

- **AutoTuner** (R package implementing the entire pipeline: TIC peak detection, EIC isolation, m/z binning with scan-continuity filtering, Gaussian-KDE fitting with KLD subsampling, and k-means clustering with outlier scoring to estimate the ppm parameter) — KujawinskiLaboratory/Autotuner
- **XCMS** (Downstream consumer — the ppm estimate feeds the centWave peak-picking algorithm that XCMS (and MZmine2) uses; the skill replaces manual tuning of one of centWave's 11 parameters)
- **R** (Implementation language; AutoTuner is distributed as an R package via BioConductor, so the k-means/KDE heuristic is invoked from R code) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
BiocManager::install("Autotuner"); # then, in R: Autotuner::estimatePpm(Autotuner::createAutotuner(filelist, sample_df))  # k-means + KDE heuristic for ppm
```

## Evaluation signals

- The returned ppm estimate equals sd(x) of the retained k-means-filtered ppm values that are all above 1, so verify no retained value ≤ 1 survives
- Cross-check that bins discarded as noise are those failing the scan-continuity criterion, not arbitrary intensity cutoffs
- Compare the ppm estimate against AutoTuner's Monte Carlo robustness expectation: estimates from random subsets of ≥9 samples should have CV < 0.1
- Confirm the KDE was subsampled until KL divergence < 0.5 rather than using the full set, and that the k-means run converged on the reduced distribution
- Validate downstream: the ppm value must be one of AutoTuner's seven continuous centWave parameter estimates written to the output file

## Limitations

- AutoTuner is not parallelized, so the KDE + k-means stage runs serially; parallelization is noted as a possible future improvement
- The heuristic is applied per sample/peak in AutoTuner's divide-and-conquer design and therefore does not compare EIC peaks across samples to estimate retention-time correction parameters
- It requires ≥3 raw data samples converted from proprietary instrument formats (mzML, mzXML, or CDF) and R ≥ 3.6; tested only on qTOF, orbitrap and Fourier transform ion cyclotron resonance mass analyzers

## Evidence

- [intro] the method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [results] bins failing the scan-continuity criterion as noise, computes ppm per bin, builds a Gaussian-KDE empirical distribution: [evidence span withheld — anchor status: not_found]
- [readme] AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF): [evidence span withheld — anchor status: external_doc]
- [discussion] For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters: "the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters"
