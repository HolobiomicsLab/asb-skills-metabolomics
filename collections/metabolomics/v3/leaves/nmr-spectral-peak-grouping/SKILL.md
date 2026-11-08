---
name: nmr-spectral-peak-grouping
description: Use when raw 1D 1H NMR spectra (spectra matrix + ppm vector) must be
  summarized into features with minimal information loss, especially when shifts or
  multiplet structure would make binning/bucketing lossy.
when_to_use_negative:
- Input is already a feature/peak-value matrix or a binned bucket table rather than
  raw spectra
- The goal is raw-spectra alignment/quantitation (speaq <= 1.2.3 CluPA/BW-ratio workflow)
  rather than peak-based processing
- Concentration estimates of known metabolites are required (e.g. targeted tools like
  BATMAN) rather than feature-based profiling
edam_operation: http://edamontology.org/operation_3255
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: speaq R package
  role: Provides getWaveletPeaks (Mexican hat wavelet peak picking), PeakGrouper (moving-window
    grouping), peak filling, and BuildFeatureMatrix
  repo: https://cran.r-project.org/package=speaq
- name: speaq 2.0
  role: The wavelet peak-picking-then-grouping method implemented in the speaq R package
  repo: https://github.com/beirnaert/speaq
- name: R
  role: Execution environment for the speaq package pipeline
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:416664c4a7ff62bffa4711b3c98aa8ec082297629989770d42c2144141bc5a93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CluPA
  - HMDBsearchR
  - R
  - icoshift
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3255
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nmr-spectral-peak-grouping

## Summary

Wavelet-based peak picking followed by moving-window peak grouping of 1D NMR spectra (speaq 2.0), which replaces binning with a peak/sample/value feature matrix to avoid the information loss of summing bins.

## When to use

Use when raw 1D 1H NMR spectra (spectra matrix + ppm vector) must be summarized into features with minimal information loss, especially when shifts or multiplet structure would make binning/bucketing lossy.

## When NOT to use

- Input is already a feature/peak-value matrix or a binned bucket table rather than raw spectra
- The goal is raw-spectra alignment/quantitation (speaq <= 1.2.3 CluPA/BW-ratio workflow) rather than peak-based processing
- Concentration estimates of known metabolites are required (e.g. targeted tools like BATMAN) rather than feature-based profiling

## Inputs

- 1D NMR spectra as a numeric matrix (rows = spectra, columns = intensity points)
- ppm vector aligned with the spectra matrix columns
- class/group factor labels for the spectra (optional, for downstream analysis)
- grouping window parameters (e.g. 0.03 ppm window, 0.015 ppm steps)

## Outputs

- Peak table from wavelet detection (peak positions/intensities per spectrum)
- Grouped peak table mapping peaks to cross-spectrum peak groups
- peak/sample/value feature matrix (e.g. 677 features for the onion intake in mice dataset)
- Silhouette diagnostics per group when SilhouetR is used

## How to apply

Load the raw spectra as a matrix with its ppm vector, then run Mexican hat wavelet peak detection (getWaveletPeaks in the speaq R package) to pick peaks from each spectrum. Group the picked peaks across spectra with the moving-window procedure (PeakGrouper), e.g. the 0.03 ppm window stepped in 0.015 ppm increments used on the onion intake in mice data, yielding a peak/sample/value feature matrix. Apply peak filling where peaks are missing from a group, and optionally run SilhouetR to detect problematic groups by silhouette values before analysis. Rationale: grouping after peak picking avoids the binning step, whose bin boundaries are hard to define in crowded spectra and which sums entire bins together, losing multiplet information (e.g. 677 speaq features vs 245 MetaboAnalyst features on the same onion peak list). The resulting feature matrix feeds directly into scaling, imputation, differential analysis, PCA, or HMDB peak queries.

## Related tools

- **speaq R package** (Provides getWaveletPeaks (Mexican hat wavelet peak picking), PeakGrouper (moving-window grouping), peak filling, and BuildFeatureMatrix) — https://cran.r-project.org/package=speaq
- **speaq 2.0** (The wavelet peak-picking-then-grouping method implemented in the speaq R package) — https://github.com/beirnaert/speaq
- **R** (Execution environment for the speaq package pipeline)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Feature matrix has a peak/sample/value structure with no binning step applied (grouping of picked peaks, not summed bins)
- On the onion intake in mice peak data with a 0.03 ppm / 0.015 ppm step moving-window grouping, the matrix contains 677 features (vs 245 for MetaboAnalyst's equivalent grouping)
- Peak filling has removed missing-peak artifacts in groups; optionally SilhouetR silhouette values do not flag problematic groups
- Downstream differential analysis (Benjamini-Hochberg FDR) and PCA run without errors on the feature matrix
- Benchmark behavior: peak-based workflow performs similar or better in ROC/Precision-Recall than binning of raw or aligned spectra (CluPA/icoshift + binning)

## Limitations

- Binning-style information loss is avoided, but grouping can still merge peaks; competitor grouping sums close peaks together, removing multiplet information (MetaboAnalyst 245 vs speaq 677 features)
- The grouping step can delete one of two large peaks, and peak filling does not always mitigate the resulting problem — detectable via the optional SilhouetR function
- Severely shifted spectra challenge alignment-based alternatives, but speaq 2.0 output quality still depends on correct wavelet peak detection

## Evidence

- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] Peak based processing (new since version 2.0): "Peak based processing (new since version 2.0)
	- peak picking (wavelets)
	- grouping
	- peak filling"
- [supplementary] grouping deletes a large peak: "The speaq 2.0 algorithm deletes one of the large peaks in the grouping step"
- [supplementary] SilhouetR detects grouping problems: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
