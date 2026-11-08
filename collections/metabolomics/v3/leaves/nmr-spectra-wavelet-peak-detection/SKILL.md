---
name: nmr-spectra-wavelet-peak-detection
description: Use when raw 1D NMR spectra (e.g. 40 wine spectra × 8712 ppm points,
  or 31 urine spectra) must be reduced to a feature matrix of peaks, avoiding binning's
  information loss and user-dependent bin boundaries.
when_to_use_negative:
- Input is already a binned bucketed table or a processed feature matrix — the wavelet
  peak picking expects raw spectra matrices
- The task is spectral alignment of severely shifted spectra rather than peak-based
  summarization — the article notes alignment algorithms are 'not always optimal when
  dealing with severely shifted spectra' and speaq 2.0 avoids binning/alignment by
  design
- Targeted concentration estimation of known metabolites is needed — the article positions
  BATMAN for that specific purpose
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides the wavelet-based peak picking (getWaveletPeaks, Mexican hat CWT)
    plus grouping and peak filling used to convert raw spectra into a feature matrix
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN-distributed R package through which speaq 2.0's peak detection, grouping,
    and matrix building functions are installed and invoked
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment for running the speaq pipeline on spectra matrices and
    writing feature matrices to CSV
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:ce399606d70ad7b60cb238910ca3648b8c0bec099b7d0584e97cb41bc6df849c
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-spectra-wavelet-peak-detection

## Summary

Wavelet-based (Mexican hat CWT) peak detection for 1D NMR spectra as implemented in speaq 2.0's getWaveletPeaks, which summarizes raw spectra into peak lists without binning and without requiring baseline correction or smoothing, feeding downstream peak grouping and peak filling.

## When to use

Use when raw 1D NMR spectra (e.g. 40 wine spectra × 8712 ppm points, or 31 urine spectra) must be reduced to a feature matrix of peaks, avoiding binning's information loss and user-dependent bin boundaries.

## When NOT to use

- Input is already a binned bucketed table or a processed feature matrix — the wavelet peak picking expects raw spectra matrices
- The task is spectral alignment of severely shifted spectra rather than peak-based summarization — the article notes alignment algorithms are 'not always optimal when dealing with severely shifted spectra' and speaq 2.0 avoids binning/alignment by design
- Targeted concentration estimation of known metabolites is needed — the article positions BATMAN for that specific purpose

## Inputs

- Raw 1D NMR spectra as a samples × measurement-points numeric matrix (e.g. 40 wine spectra × 8712 points)
- Matching ppm axis vector (e.g. Winedata$ppm)
- Optional class factor for visualization/grouping (e.g. wine.color)

## Outputs

- Per-spectrum peak list (ppm position, intensity, sample index) from getWaveletPeaks
- Grouped peak table after PeakGrouper
- Feature/sample/peak-value matrix after BuildFeatureMatrix (e.g. 40×207 wine, 31×677 onion)

## How to apply

Load the spectra as a matrix with a matching ppm vector in R (e.g. Winedata$spectra with Winedata$ppm). Run speaq 2.0's Mexican hat continuous-wavelet peak detection via getWaveletPeaks on each spectrum; the wavelet formulation removes the need for baseline correction and smoothing pre-steps. Detection runs in parallel by default and outputs a peak table (ppm positions, intensities, sample indices) rather than binned buckets. Feed the peak table directly into PeakGrouper to form cross-spectrum peak groups, then BuildFeatureMatrix to obtain the sample-by-peak feature matrix — e.g. 40×207 for the wine data or 31×677 for the onion intake in mice data. Validate by checking the reduced dimensionality and confirming that downstream statistics (differential analysis, PCA) behave sensibly on the resulting matrix.

## Related tools

- **speaq 2.0** (Provides the wavelet-based peak picking (getWaveletPeaks, Mexican hat CWT) plus grouping and peak filling used to convert raw spectra into a feature matrix) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN-distributed R package through which speaq 2.0's peak detection, grouping, and matrix building functions are installed and invoked) — https://cran.r-project.org/package=speaq
- **R** (Execution environment for running the speaq pipeline on spectra matrices and writing feature matrices to CSV)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Feature matrix has the expected reduced dimensions: 40 samples × 207 peak groups for the wine dataset, 31 samples × 677 features for the onion intake in mice dataset
- Peak detection completes without a prior baseline-correction or smoothing step, per the wavelet proposition
- Bonferroni-corrected linear-model differential analysis on the onion feature matrix recovers exactly 9 significant peaks, matching Fig 8 of the paper
- PCA score plot on the wine feature matrix shows PC2 separating red, white and rosé wines as in the supplementary figure
- Number of detected peaks per spectrum is stable across reruns (parallel default mode should not change peak counts)

## Limitations

- In some cases the grouping step deletes one of the large peaks, and the peak filling step does not always mitigate the resulting missing feature; the optional SilhouetR function detects this problem via per-group silhouette values
- The peak-based approach deliberately avoids binning and spectral alignment, so it is not a substitute for alignment algorithms when alignment itself is the analysis goal
- Detector performance claims are benchmarked on a simulated dataset and two public datasets, not exhaustively across all NMR acquisition conditions

## Evidence

- [abstract] uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction: "uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction"
- [abstract] peak-picking followed by grouping, avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README minimal example calling getWaveletPeaks: [evidence span withheld — anchor status: not_found]
- [readme] README peak-based feature list: "peak picking (wavelets)
	- grouping
	- peak filling
	- scaling & imputations"
- [supplementary] SilhouetR detects grouping failures: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
