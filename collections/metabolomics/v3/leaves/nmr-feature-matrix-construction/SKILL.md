---
name: nmr-feature-matrix-construction
description: Use when raw 1D NMR spectra must be reduced to a feature matrix of peak
  groups for downstream statistics (PCA, differential analysis), and binning would
  lose information.
when_to_use_negative:
- Input is already a feature table or peak-value matrix (e.g. post-binning data) rather
  than raw spectra
- Severely shifted spectra where alignment-based approaches are the explicit target
  — the supplementary notes spectral alignment algorithms 'is not always optimal when
  dealing with severely shifted spectra', so peak-based processing should not be conflated
  with alignment methods
- Targeted quantification of known metabolite concentrations, where tools like BATMAN
  (aimed at concentration estimates for known metabolites) are the appropriate choice
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides the wavelet-based peak picking, grouping, peak filling and feature-matrix
    construction workflow
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN-distributed R package exposing getWaveletPeaks, PeakGrouper and BuildFeatureMatrix
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq package functions are invoked
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:380187a665eafb2db6d7d2f399fba111781f0f68eeadc9b6f9b1d32a40870f20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nmr-feature-matrix-construction

## Summary

Constructs a feature/sample/peak-value matrix from raw 1D NMR spectra using the speaq 2.0 wavelet-based pipeline (peak picking with the Mexican hat CWT, peak grouping, and peak filling), replacing binning to avoid information loss. Applied to raw spectral matrices (e.g. 40 wines x 8712 points, 31 urine spectra) it reduces them to compact peak-group feature tables (e.g. 40 x 207, 31 x 677).

## When to use

Use when raw 1D NMR spectra must be reduced to a feature matrix of peak groups for downstream statistics (PCA, differential analysis), and binning would lose information.

## When NOT to use

- Input is already a feature table or peak-value matrix (e.g. post-binning data) rather than raw spectra
- Severely shifted spectra where alignment-based approaches are the explicit target — the supplementary notes spectral alignment algorithms 'is not always optimal when dealing with severely shifted spectra', so peak-based processing should not be conflated with alignment methods
- Targeted quantification of known metabolite concentrations, where tools like BATMAN (aimed at concentration estimates for known metabolites) are the appropriate choice

## Inputs

- Raw 1D NMR spectra matrix (samples x measurement points), e.g. 40 wine spectra x 8712 points
- ppm vector of measurement point positions
- Optional class/group factor per spectrum (e.g. wine.color, onion diet percentage)

## Outputs

- Feature/sample/peak-value matrix (samples x peak groups), e.g. 40 x 207 for wine data or 31 x 677 for onion-intake mouse urine data
- Peak list with detected peaks per spectrum
- Peak groups table from the grouping step

## How to apply

Load the raw spectra as a matrix plus an ppm vector (e.g. Winedata$spectra and Winedata$ppm in R with the speaq package). Detect peaks in each spectrum with getWaveletPeaks, which uses the Mexican hat continuous wavelet transform and avoids the need for baseline correction and smoothing. Group detected peaks across spectra with PeakGrouper to form peak groups; optionally use the SilhouetR function to compute silhouette values per group and flag problematic groups. Apply peak filling (via BuildFeatureMatrix) to recover peak values in groups where a peak was missed, yielding the feature/sample/peak-value matrix. Verify the resulting dimensions match expectations (e.g. 40 samples x 207 peak groups for the wine dataset; 31 samples x 677 features for the onion-intake-in-mice dataset) before passing the matrix to scaling, transformation, PCA or linear-model differential analysis.

## Related tools

- **speaq 2.0** (Provides the wavelet-based peak picking, grouping, peak filling and feature-matrix construction workflow) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN-distributed R package exposing getWaveletPeaks, PeakGrouper and BuildFeatureMatrix) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq package functions are invoked)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Resulting feature matrix has expected dimensions: 40 samples x 207 peak groups for the wine dataset; 31 samples x 677 features for the onion-intake mouse urine dataset
- Number of detected peak groups is far smaller than the original measurement points (207 << 8712), confirming dimension reduction without binning
- Silhouette values computed by the optional SilhouetR function per group can be inspected to detect problematic peak groups
- Downstream PCA on the feature matrix should reproduce expected patterns, e.g. Principal Component 2 separating red, white and rosé wines
- Feature matrix is free of missing peak values after the peak filling step recovers missed peaks

## Limitations

- In some cases a large peak is deleted in the grouping step, and the peak filling step does not always mitigate the resulting problem
- Peak-based processing replaces binning but is a distinct paradigm from spectral alignment; results of alignment algorithms are not always optimal for severely shifted spectra, and the two approaches address different issues
- Baseline correction and smoothing are intentionally avoided in the peak detection step, so the method assumes the wavelet approach is robust to uncorrected baselines

## Evidence

- [abstract] uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction: "uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction"
- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] peak picking (wavelets), grouping, peak filling, scaling & imputations: "	- peak picking (wavelets)
	- grouping
	- peak filling
	- scaling & imputations"
- [readme] peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector): [evidence span withheld — anchor status: not_found]
- [supplementary] This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group.: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [intro] The basic proposition of speaq 2.0 is to use wavelets to summarize the peaks within the spectra.: "The basic proposition of speaq 2.0 is to use wavelets to summarize the peaks within the spectra."
