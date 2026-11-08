---
name: r-based-nmr-data-processing
description: Use when processing 1D NMR spectra in R (peak picking, grouping, feature-matrix
  construction) or when peak groups look wrong and you need silhouette-based detection
  plus regroupR correction on a dataset like the wine spectra.
when_to_use_negative:
- Input is already a feature table or binned/bucketed spectra — speaq 2.0's purpose
  is to avoid the binning step by working from peak lists
- The goal is metabolite concentration estimates for known metabolites only — tools
  like BATMAN are aimed at that specific purpose
- Raw spectra only need baseline correction or alignment without peak-based processing
  — the speaq <= 1.2.3 raw-spectra route (CluPA alignment) is the appropriate path
edam_operation: http://edamontology.org/operation_3432
tools:
- name: speaq 2.0
  role: 'Complete 1D NMR workflow: wavelet peak picking (getWaveletPeaks), grouping
    (PeakGrouper), silhouette-based error detection (SilhouetR), and regrouping (regroupR)'
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN distribution through which speaq 2.0 functions (SilhouetR, regroupR,
    getWaveletPeaks, PeakGrouper) are installed and loaded
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq package functions are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:eba32231d30c805aed83c8b08c840605489a9f33af247c44cc3438751d0ad3ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# r-based-nmr-data-processing

## Summary

A peak-based 1D NMR spectra processing workflow in R via the speaq 2.0 package (wavelet peak picking, grouping, peak filling, and silhouette-based detection and correction of misgrouped peaks), avoiding the information loss inherent to binning.

## When to use

Use when processing 1D NMR spectra in R (peak picking, grouping, feature-matrix construction) or when peak groups look wrong and you need silhouette-based detection plus regroupR correction on a dataset like the wine spectra.

## When NOT to use

- Input is already a feature table or binned/bucketed spectra — speaq 2.0's purpose is to avoid the binning step by working from peak lists
- The goal is metabolite concentration estimates for known metabolites only — tools like BATMAN are aimed at that specific purpose
- Raw spectra only need baseline correction or alignment without peak-based processing — the speaq <= 1.2.3 raw-spectra route (CluPA alignment) is the appropriate path

## Inputs

- 1D NMR spectra as a numeric matrix (rows = samples, columns = spectral points), e.g. Winedata$spectra
- Matching ppm vector for the spectra, e.g. Winedata$ppm
- Optional class factor for grouping samples, e.g. Winedata$wine.color (red/white/rosé)
- Peak-picked and grouped peak data from speaq (output of getWaveletPeaks and PeakGrouper) for SilhouetR/regroupR

## Outputs

- Peak list from wavelet peak detection (ppm positions, intensities, SNR)
- Grouped peak assignments per sample from PeakGrouper
- Feature/sample/peak-value matrix from BuildFeatureMatrix
- Mean silhouette values per peak group from SilhouetR
- Corrected peak group assignments with recomputed silhouette values (e.g. written to wine_regrouped_groups.csv)

## How to apply

Install speaq from CRAN (or GitHub via devtools), then run the peak-based pipeline: build the spectra matrix and ppm vector from the input (e.g. Winedata$spectra and Winedata$ppm), pick peaks with getWaveletPeaks (Mexican hat wavelets, default parallel mode), and group them with PeakGrouper to obtain the feature/sample/peak-value matrix via BuildFeatureMatrix, skipping the binning step. To detect grouping errors, run SilhouetR to compute mean silhouette values per peak group across samples; flag groups with a low mean silhouette (on the wine data the misgrouped group averaged ~0.25) despite fairly high average SNR, since high SNR rules out noise as the cause. Correct flagged groups by submitting them to regroupR, which computes the initial between-peak distance using both ppm values and peak signal-to-noise ratio before reassigning peaks to better-fitting existing groups. Validate by recomputing silhouette values: the previously flagged group should show an improved mean silhouette after regrouping, confirming the correction.

## Related tools

- **speaq 2.0** (Complete 1D NMR workflow: wavelet peak picking (getWaveletPeaks), grouping (PeakGrouper), silhouette-based error detection (SilhouetR), and regrouping (regroupR)) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN distribution through which speaq 2.0 functions (SilhouetR, regroupR, getWaveletPeaks, PeakGrouper) are installed and loaded) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq package functions are executed)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Flagged misgrouped group has a low mean silhouette value (~0.25 on the wine data) but fairly high average SNR before correction
- After regroupR, the previously low-silhouette group shows an improved mean silhouette value, confirming the correction
- The feature matrix from BuildFeatureMatrix has one entry per (sample, peak group) pair with no binning artifacts
- Pipeline runs end-to-end from spectra matrix + ppm vector through getWaveletPeaks, PeakGrouper, and BuildFeatureMatrix without errors
- Downstream analysis (e.g. PCA on the feature matrix) separates known sample classes such as red, white and rosé wines

## Limitations

- Grouping errors can occur (e.g. the algorithm deletes one of two large peaks in a grouping step, and peak filling may not always mitigate it), which is why SilhouetR checking is needed
- SilhouetR and regroupR correction is optional and must be run deliberately; grouping errors are otherwise not automatically surfaced
- Severely shifted spectra can also challenge spectral alignment approaches, which the peak-based method is meant to circumvent but does not eliminate all preprocessing issues

## Evidence

- [supplementary] SilhouetR detects misgrouped peaks via silhouette values: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [abstract] speaq 2.0 distributed as R package: "speaq 2.0 is distributed through the existing speaq R package"
- [readme] Peak-based workflow steps in README: "* Peak based approach (new since v2.0)
	- peak picking (wavelets)
	- grouping
	- peak filling"
- [abstract] Workflow avoids binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README example pipeline code: "peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)  

# peak grouping
groups <- PeakGrouper(Y.peaks = peaks)"
