---
name: snr-based-peak-quality-evaluation
description: Use when peak-picked and grouped 1D NMR spectra show suspicious peak
  groups (e.g., peaks from different metabolites merged), and you need to detect misgrouping
  via silhouette values plus SNR before regrouping.
when_to_use_negative:
- Input is raw spectra that have not yet been peak-picked and grouped by getWaveletPeaks/PeakGrouper
  — silhouette evaluation operates on peak groups, not raw spectra
- Groups already show high mean silhouette values — regrouping is unnecessary and
  the peak-filling step alone may mitigate the issue
tools:
- name: speaq 2.0
  role: Provides SilhouetR for per-group silhouette calculation and regroupR for SNR-
    and ppm-based correction of misgrouped peaks
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R distribution of speaq 2.0 functions including peak picking (getWaveletPeaks),
    grouping (PeakGrouper) and the optional SilhouetR quality check
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment for running the speaq workflow and writing corrected
    group assignments to CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:4475cce86ea5f1a51bd38f04b82d5ff501b961e9f7cee6dffd70162b6431705c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# snr-based-peak-quality-evaluation

## Summary

Evaluate peak-group quality in speaq 2.0 by computing per-group mean silhouette values with SilhouetR and cross-checking them against peak signal-to-noise ratios, so that grouping errors (low silhouette despite high SNR) can be flagged and then fixed with regroupR.

## When to use

Use when peak-picked and grouped 1D NMR spectra show suspicious peak groups (e.g., peaks from different metabolites merged), and you need to detect misgrouping via silhouette values plus SNR before regrouping.

## When NOT to use

- Input is raw spectra that have not yet been peak-picked and grouped by getWaveletPeaks/PeakGrouper — silhouette evaluation operates on peak groups, not raw spectra
- Groups already show high mean silhouette values — regrouping is unnecessary and the peak-filling step alone may mitigate the issue

## Inputs

- Peak-picked and grouped 1D NMR spectra from the speaq 2.0 workflow (e.g., Winedata$spectra with Winedata$ppm vector)
- Peak groups produced by PeakGrouper (per-group peak assignments with ppm values and SNR)

## Outputs

- Per-peak-group mean silhouette values (SilhouetR output)
- Per-group average peak signal-to-noise ratio
- Corrected group assignments after regroupR(), with recomputed silhouette values (e.g., written to wine_regrouped_groups.csv)

## How to apply

Start from the peak-picked and grouped spectra produced by speaq 2.0's wavelet peak picking (getWaveletPeaks) and PeakGrouper on 1D NMR data such as the Winedata spectra. Run the optional SilhouetR function to calculate silhouette values for each peak group across samples, then compute each group's average peak signal-to-noise ratio. Flag groups with low mean silhouette values (in the wine dataset the problematic group had a mean silhouette of ~0.25) that nonetheless have fairly high average SNR — this combination indicates a genuine misgrouping rather than a noise artifact. Correct flagged groups by submitting them to regroupR(), which recalculates between-peak distances using both ppm values and peak SNR before reassigning peaks to better-fitting existing groups. Validate by recomputing silhouette values: the corrected group should show an improved mean silhouette after regrouping.

## Related tools

- **speaq 2.0** (Provides SilhouetR for per-group silhouette calculation and regroupR for SNR- and ppm-based correction of misgrouped peaks) — https://github.com/beirnaert/speaq
- **speaq R package** (R distribution of speaq 2.0 functions including peak picking (getWaveletPeaks), grouping (PeakGrouper) and the optional SilhouetR quality check) — https://cran.r-project.org/package=speaq
- **R** (Execution environment for running the speaq workflow and writing corrected group assignments to CSV)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); sil <- SilhouetR(Y.peaks = groups)
```

## Evaluation signals

- The flagged group has a low mean silhouette value (~0.25 in the wine data) combined with fairly high average SNR
- After regroupR(), the previously flagged group's mean silhouette value improves relative to its pre-regrouping value
- Corrected group assignments are written out (e.g., wine_regrouped_groups.csv) with recomputed silhouette values
- Remaining groups retain stable group memberships (no unintended reassignment of well-formed groups)

## Limitations

- The grouping step can delete one of the large peaks, a problem usually mitigated by the peak filling step but not always
- SilhouetR is an optional step; without it, misgroupings may go undetected in the grouped peak matrix
- Regrouping requires both ppm-distance and SNR-distance information, so peaks lacking reliable SNR estimates cannot be confidently reassigned

## Evidence

- [supplementary] SilhouetR detects misgrouping via per-group silhouette values: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [supplementary] Peak filling step does not always mitigate the grouping problem: "This problem is usually mitigated by the peak filling step but in this case it is not."
- [supplementary] Grouping step can delete large peaks: "The speaq 2.0 algorithm deletes one of the large peaks in the grouping step"
- [readme] Peak-based processing pipeline feeding the quality evaluation: "# peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)  

# peak grouping
groups <- PeakGrouper(Y.peaks = peaks)"
