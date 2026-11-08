---
name: nmr-spectra-grouping-correction
description: Use when peak-picked, already-grouped NMR spectra contain suspicious
  peak groups — e.g., a group with a low mean silhouette value (~0.25) but a high
  average SNR — and the groups must be reassigned before building the feature matrix.
when_to_use_negative:
- Input is raw, un-peak-picked spectra — run the wavelet peak-picking (getWaveletPeaks)
  and PeakGrouper steps first
- The problem is spectral shift/alignment rather than peak grouping errors — use the
  CluPA alignment functionality instead
- Input is already a validated feature matrix with no suspicious groups; silhouette
  screening would add no correction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: speaq 2.0
  role: Provides SilhouetR (silhouette-based detection of misgrouped peak groups)
    and regroupR (regrouping using ppm and SNR distances)
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributed R package supplying getWaveletPeaks, PeakGrouper, SilhouetR and
    regroupR functions and the wine example data
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment for running the speaq pipeline and regrouping correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:7307afdb4586f7cbd128d6814fd8fcbd6efdad4e27a74a849098b533ae32f9a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nmr-spectra-grouping-correction

## Summary

Detects incorrectly grouped peaks in 1D NMR peak-based workflows using per-group mean silhouette values (SilhouetR), then corrects misgrouped peak groups with speaq 2.0's regroupR(), which re-clusters peaks using ppm distance and peak SNR. Validated on the speaq wine dataset.

## When to use

Use when peak-picked, already-grouped NMR spectra contain suspicious peak groups — e.g., a group with a low mean silhouette value (~0.25) but a high average SNR — and the groups must be reassigned before building the feature matrix.

## When NOT to use

- Input is raw, un-peak-picked spectra — run the wavelet peak-picking (getWaveletPeaks) and PeakGrouper steps first
- The problem is spectral shift/alignment rather than peak grouping errors — use the CluPA alignment functionality instead
- Input is already a validated feature matrix with no suspicious groups; silhouette screening would add no correction

## Inputs

- Peak-picked spectra from getWaveletPeaks (Y.peaks data frame with ppm and SNR per peak)
- Grouped peaks output from PeakGrouper (group assignments per peak)
- Raw spectra matrix and ppm vector (e.g., Winedata$spectra, Winedata$ppm)
- Sample class factor (e.g., Winedata$wine.color)

## Outputs

- Mean silhouette value per peak group (from SilhouetR)
- Corrected peak group assignments from regroupR()
- Recomputed silhouette values for corrected groups (e.g., wine_regrouped_groups.csv)

## How to apply

Load the peak-picked and grouped spectra (in the wine case, the Winedata spectra matrix, ppm vector, and wine.color class factor from the speaq R package). Run SilhouetR from speaq 2.0 to compute the mean silhouette value for each peak group across samples; flag groups with low mean silhouette despite high average SNR as misgrouping candidates (the wine data contains such a group with mean silhouette ~0.25). Submit the wrong groups to regroupR(), which uses both ppm values and peak signal-to-noise ratio for the initial between-peak distance calculation before regrouping peaks into better-fitting existing groups. Recompute silhouette values on the corrected assignments to confirm the flagged group improved, and export the corrected groups (e.g., to wine_regrouped_groups.csv). Rationale: the automatic PeakGrouper grouping can fail on certain peaks, and this detection is 'mitigated' neither by peak filling nor by alignment, so an explicit silhouette-based check plus regroupR correction is required.

## Related tools

- **speaq 2.0** (Provides SilhouetR (silhouette-based detection of misgrouped peak groups) and regroupR (regrouping using ppm and SNR distances)) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributed R package supplying getWaveletPeaks, PeakGrouper, SilhouetR and regroupR functions and the wine example data) — https://cran.r-project.org/package=speaq
- **R** (Execution environment for running the speaq pipeline and regrouping correction)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks)
```

## Evaluation signals

- The flagged group shows a low mean silhouette (~0.25 on wine data) while its average SNR is fairly high — this combination identifies the misgrouping
- After regroupR(), the previously low-silhouette group's mean silhouette value improves, confirming the correction
- regroupR uses both ppm values and peak SNR for between-peak distances, so reassigned peaks must be closer in both ppm and SNR to their new group
- Corrected group assignments and recomputed silhouette values are exported and inspectable (wine_regrouped_groups.csv)
- Downstream feature matrix (BuildFeatureMatrix on the corrected groups) remains consistent in dimensions with the number of samples and groups

## Limitations

- SilhouetR is an optional detection function; without it, grouping errors can go unnoticed because the problem 'is usually mitigated by the peak filling step but in this case it is not'
- Detection relies on the low-silhouette/high-SNR signature — misgroupings without this pattern may not be flagged
- Grouping is downstream of wavelet peak picking; errors in peak picking (e.g., deletion of a large peak in the grouping step) may require other mitigations

## Evidence

- [supplementary] SilhouetR detects misgrouped groups via silhouette values: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [supplementary] Peak filling does not mitigate this grouping problem: "This problem is usually mitigated by the peak filling step but in this case it is not."
- [readme] Peak picking and grouping steps in the speaq README pipeline: "# peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)

# peak grouping
groups <- PeakGrouper(Y.peaks = peaks)"
- [abstract] speaq 2.0 workflow: peak-picking then grouping, avoiding binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
