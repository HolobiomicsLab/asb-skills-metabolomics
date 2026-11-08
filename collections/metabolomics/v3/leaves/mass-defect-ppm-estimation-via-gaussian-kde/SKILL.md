---
name: mass-defect-ppm-estimation-via-gaussian-kde
description: Use when raw scan-level mass-spectral data (m/z, intensity, retention
  time per scan) and detected TIC peak boundaries are available and a dataset-specific
  centWave ppm parameter must be estimated for XCMS/MZmine2 peak picking, without
  iterative parameter sweeping.
when_to_use_negative:
- Input is already a feature table (post peak-picking) rather than raw scan-level
  data — the EIC-within-TIC-peak statistics require raw scans
- The target parameter is mzDiff or a non-continuous centWave parameter — AutoTuner
  optimizes seven continuous parameters plus one grouping parameter and does not estimate
  mzDiff (only a sensitivity analysis showed distinct mzDiff values had minimal effect
  on the returned feature table)
- Fewer than 3 raw samples are available (AutoTuner README input requirement)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: AutoTuner
  role: Implements the EIC binning, scan-continuity filtering, Gaussian-KDE fitting
    with KLD<0.5 subsampling, and k-means/outlier-score selection that produce the
    ppm estimate
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: XCMS
  role: Downstream consumer — the estimated ppm is one of the seven continuous centWave
    parameters used for XCMS peak picking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:2370620780389d234f03a8ac99fd49f8a417ce234a27e299c08396f0188e8345
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-defect-ppm-estimation-via-gaussian-kde

## Summary

AutoTuner's single-step statistical-inference method for estimating the centWave ppm (m/z tolerance) parameter: it fits a Gaussian kernel-density empirical distribution to per-bin ppm deviations computed from EICs within each detected TIC peak, then separates true-feature ppm values from noise-derived ppm values via k-means clustering with an outlier score, returning the sd of the retained values as the ppm estimate.

## When to use

Use when raw scan-level mass-spectral data (m/z, intensity, retention time per scan) and detected TIC peak boundaries are available and a dataset-specific centWave ppm parameter must be estimated for XCMS/MZmine2 peak picking, without iterative parameter sweeping.

## When NOT to use

- Input is already a feature table (post peak-picking) rather than raw scan-level data — the EIC-within-TIC-peak statistics require raw scans
- The target parameter is mzDiff or a non-continuous centWave parameter — AutoTuner optimizes seven continuous parameters plus one grouping parameter and does not estimate mzDiff (only a sensitivity analysis showed distinct mzDiff values had minimal effect on the returned feature table)
- Fewer than 3 raw samples are available (AutoTuner README input requirement)

## Inputs

- Raw scan-level mass-spectral data (m/z, intensity, retention time per scan) from mzML, mzXML, or CDF files
- Detected TIC peak boundaries (retention-time windows) from AutoTuner's TIC peak detection step
- Sample metadata spreadsheet with at least two columns (sample name, experimental factor) — at least 3 raw samples required

## Outputs

- A scalar ppm (m/z tolerance) estimate — the sd of the k-means-retained true-feature ppm values
- Intermediate per-bin ppm values and the Gaussian-KDE empirical distribution of ppm deviations

## How to apply

For each detected TIC peak, isolate the EICs of ions eluting within that peak's retention-time window. Within each EIC, sort and bin the m/z values and discard bins that fail the scan-continuity criterion, since these are noise-derived. Compute a ppm deviation value per surviving bin, then build a Gaussian kernel-density empirical distribution of the ppm values, subsampling first until the Kullback-Leibler divergence between subsample and full distribution falls below 0.5. Run k-means clustering on the KDE output and apply an outlier-score heuristic to retain only true-feature ppm values (values above 1); the final ppm estimate is sd(x) of those retained values. Rationale: this performs statistical inference on raw data in one step rather than iteratively checking parameter values, giving the ~100-1000X speed improvement over sweep-based optimizers like IPO.

## Related tools

- **AutoTuner** (Implements the EIC binning, scan-continuity filtering, Gaussian-KDE fitting with KLD<0.5 subsampling, and k-means/outlier-score selection that produce the ppm estimate) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **XCMS** (Downstream consumer — the estimated ppm is one of the seven continuous centWave parameters used for XCMS peak picking)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- The retained ppm values are all above 1 (values at or below 1 are the outlier-score-rejected noise-derived ppm values)
- Subsampling terminates only when the KLD between the subsample's KDE and the full distribution is < 0.5
- The returned ppm estimate equals sd(x) over the retained true-feature ppm values, not the raw per-bin mean
- The estimated ppm lies in a plausible instrument range for qTOF/orbitrap/FT-ICR data and reproduces across random sample subsets (Monte Carlo robustness; CV < 0.1 with ~9 samples)
- Bins failing the scan-continuity criterion are excluded before KDE fitting — verify noise bins do not enter the distribution

## Limitations

- The ppm estimate depends on upstream TIC peak detection quality; the sliding window TIC analysis requires user input in the first portion of AutoTuner
- AutoTuner's divide-and-conquer approach does not compare EIC peaks across samples, so no retention-time correction information is available at this stage
- Parameter estimates were validated as most-robust with 9-12 samples (cultures vs communities); fewer samples risk unstable estimates
- AutoTuner is not parallelized, and tested only on untargeted qTOF, orbitrap and FT-ICR data per the README

## Evidence

- [intro] AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [intro] The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values.: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [abstract] it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes): "it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes)."
- [readme] AutoTuner is used to identify dataset specific parameters to process untargeted metabolomics data.: [evidence span withheld — anchor status: external_doc]
