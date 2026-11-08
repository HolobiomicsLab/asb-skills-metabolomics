---
name: cross-spectrum-peak-grouping
description: Use when multiple 1D NMR spectra (e.g. 40 wine spectra of 8712 points,
  or 31 mouse-urine spectra) need their per-spectrum wavelet peaks matched across
  samples to form a shared feature matrix, avoiding binning.
when_to_use_negative:
- The input is already a binned/bucketed or otherwise summarized feature table rather
  than raw spectra with per-spectrum peaks
- Only a single spectrum is available, so there is no cross-spectrum grouping to perform
- The goal is spectral alignment of severely shifted raw spectra rather than peak-based
  feature extraction — alignment algorithms may be suboptimal and speaq 2.0's peak-based
  route avoids binning instead
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides the peak picking, grouping and peak filling workflow that implements
    cross-spectrum peak grouping
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributes getWaveletPeaks, PeakGrouper and BuildFeatureMatrix used to detect,
    group and fill peaks across spectra
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment in which the speaq R package grouping functions are
    invoked
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:92f5216b5afdfeb64b4cf06c871d1dbc5953057bd4ad7823a069ab2a6df12675
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-spectrum-peak-grouping

## Summary

The speaq 2.0 peak-grouping step aligns wavelet-detected peaks across multiple 1D NMR spectra into common peak groups, replacing binning/bucketing so that a feature/sample/peak-value matrix can be built without bin-boundary information loss.

## When to use

Use when multiple 1D NMR spectra (e.g. 40 wine spectra of 8712 points, or 31 mouse-urine spectra) need their per-spectrum wavelet peaks matched across samples to form a shared feature matrix, avoiding binning.

## When NOT to use

- The input is already a binned/bucketed or otherwise summarized feature table rather than raw spectra with per-spectrum peaks
- Only a single spectrum is available, so there is no cross-spectrum grouping to perform
- The goal is spectral alignment of severely shifted raw spectra rather than peak-based feature extraction — alignment algorithms may be suboptimal and speaq 2.0's peak-based route avoids binning instead

## Inputs

- Matrix of raw 1D NMR spectra (samples x ppm measurement points), e.g. Winedata$spectra (40 spectra x 8712 points)
- PPM vector corresponding to the spectral axis (e.g. Winedata$ppm)
- Per-spectrum peak lists produced by speaq 2.0 wavelet peak picking (getWaveletPeaks output)

## Outputs

- Peak-group table grouping detected peaks across spectra by ppm position
- Feature/sample/peak-value matrix after peak filling (e.g. 40 samples x 207 peak groups for wine; 31 x 677 for onion intake in mice)
- Optional SilhouetR silhouette values per peak group for quality control

## How to apply

First run speaq 2.0's wavelet-based peak picking (Mexican hat CWT, via getWaveletPeaks / detectSpecPeaks) on each spectrum individually, deliberately skipping baseline correction and smoothing since the wavelet step tolerates raw spectra. Then feed the per-spectrum peak lists into the grouping step (PeakGrouper), which matches peaks at similar ppm positions across all spectra to form peak groups; when a peak is absent in some spectra, the peak filling step recovers its value so the matrix stays complete. Optionally run SilhouetR, which calculates silhouette values for each group, to detect problematic groups (e.g. where a large peak was incorrectly deleted during grouping). The result is a feature/sample/peak-value matrix (e.g. 40 x 207 for the wine data, 31 x 677 for the onion intake in mice data) that feeds downstream scaling, imputation, PCA, or Bonferroni-corrected linear-model differential analysis.

## Related tools

- **speaq 2.0** (Provides the peak picking, grouping and peak filling workflow that implements cross-spectrum peak grouping) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributes getWaveletPeaks, PeakGrouper and BuildFeatureMatrix used to detect, group and fill peaks across spectra) — https://cran.r-project.org/package=speaq
- **R** (Execution environment in which the speaq R package grouping functions are invoked) — https://cran.r-project.org/package=speaq

## Examples

```
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The grouped-and-filled feature matrix has the expected dimensions reported in the paper: 40 samples x 207 peak groups for the wine data and 31 samples x 677 features for the onion intake in mice data
- SilhouetR silhouette values per group do not flag groups where a large peak was incorrectly deleted or merged
- Peak filling has recovered missing peak values so no group column is left empty across samples
- Downstream PCA on the feature matrix behaves as expected (e.g. wine color separation along PC2; no consistent diet-group separation in the onion study)
- Differential analysis on the grouped matrix reproduces the reported 9 Bonferroni-significant peaks for the onion intake dataset (Fig 8)

## Limitations

- The grouping algorithm can delete one of the large peaks in the grouping step, producing a bad group; this is usually mitigated by the peak filling step but not always, and detection relies on the optional SilhouetR function
- Avoiding the binning step means group quality depends on the wavelet peak picking; no baseline correction or smoothing is performed beforehand
- Spectral alignment algorithms do not always perform optimally on severely shifted spectra, motivating the peak-based alternative but not guaranteeing perfect grouping

## Evidence

- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] SilhouetR detects problematic groups: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [supplementary] grouping deletes large peak; peak filling mitigates: "The speaq 2.0 algorithm deletes one of the large peaks in the grouping step"
- [supplementary] peak filling step usually mitigates the problem: "This problem is usually mitigated by the peak filling step but in this case it is not."
- [readme] README minimal example with PeakGrouper: [evidence span withheld — anchor status: not_found]
