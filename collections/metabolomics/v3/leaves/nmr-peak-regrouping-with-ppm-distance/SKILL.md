---
name: nmr-peak-regrouping-with-ppm-distance
description: Use when peak grouping of 1D NMR spectra (e.g., the wine dataset processed
  with speaq 2.0) has produced suspect groups, indicated by a low mean silhouette
  value despite a high average peak SNR.
when_to_use_negative:
- The input is raw unprocessed spectra with no peak picking or grouping yet performed
  (start with getWaveletPeaks and PeakGrouper instead)
- A group has a low mean silhouette AND low average SNR — the wine-data diagnostic
  relies on low silhouette with fairly high SNR to distinguish misgrouping from noise
- The goal is raw spectra alignment or quantitation via speaq <= 1.2.3 (CluPA / BW-ratio),
  which is a separate peak-based-processing workflow
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides SilhouetR for silhouette-based group quality scoring and regroupR
    for ppm- and SNR-aware peak reassignment
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN distribution through which SilhouetR and regroupR are invoked in R; also
    ships the wine data
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment for running the silhouette scoring and regrouping functions
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:3a3ca271289a57a4efbaaf0b5662321713fc213c9ff61cacc2e0bcb2a81c0c06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nmr-peak-regrouping-with-ppm-distance

## Summary

Detect and correct incorrectly grouped NMR peaks in the speaq 2.0 workflow by scoring each peak group with silhouette values (SilhouetR) and reassigning misgrouped peaks with regroupR, which combines ppm distance and peak signal-to-noise ratio. This restores group purity when the peak grouper has merged peaks that belong in separate groups.

## When to use

Use when peak grouping of 1D NMR spectra (e.g., the wine dataset processed with speaq 2.0) has produced suspect groups, indicated by a low mean silhouette value despite a high average peak SNR.

## When NOT to use

- The input is raw unprocessed spectra with no peak picking or grouping yet performed (start with getWaveletPeaks and PeakGrouper instead)
- A group has a low mean silhouette AND low average SNR — the wine-data diagnostic relies on low silhouette with fairly high SNR to distinguish misgrouping from noise
- The goal is raw spectra alignment or quantitation via speaq <= 1.2.3 (CluPA / BW-ratio), which is a separate peak-based-processing workflow

## Inputs

- Peak-picked and grouped 1D NMR spectra data (e.g., the wine data from the speaq R package)
- Per-group mean silhouette values from SilhouetR
- Peak signal-to-noise ratio (SNR) values for peaks in flagged groups
- ppm vector for the spectra (e.g., Winedata$ppm)

## Outputs

- Corrected peak group assignments from regroupR()
- Recomputed per-group mean silhouette values after regrouping
- CSV of corrected group assignments (e.g., wine_regrouped_groups.csv)

## How to apply

After peak picking (getWaveletPeaks) and grouping (PeakGrouper) on the wine spectra, run SilhouetR from speaq 2.0 to compute a mean silhouette value per peak group across samples. Flag groups with a low mean silhouette (e.g., ~0.25 on the wine data) that nonetheless have a fairly high average SNR, as this combination signals a grouping error rather than a noisy peak. Submit the wrong group(s) to regroupR(), which computes between-peak distances using both ppm values and peak signal-to-noise ratio before reassigning peaks to better-fitting existing groups. Recompute silhouette values on the corrected grouping and write the corrected assignments out (e.g., wine_regrouped_groups.csv). Validation: the previously flagged group's mean silhouette improves after regrouping.

## Related tools

- **speaq 2.0** (Provides SilhouetR for silhouette-based group quality scoring and regroupR for ppm- and SNR-aware peak reassignment) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN distribution through which SilhouetR and regroupR are invoked in R; also ships the wine data) — https://cran.r-project.org/package=speaq
- **R** (Execution environment for running the silhouette scoring and regrouping functions) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks)
```

## Evaluation signals

- The flagged group's mean silhouette value (e.g., ~0.25 pre-correction on the wine data) increases after regroupR is applied
- Silhouette values are recomputed for all groups and no previously high-silhouette group drops substantially
- The corrected grouping is written to wine_regrouped_groups.csv with complete group assignments for all peaks
- regroupR's distance calculation used both ppm values and peak SNR, confirmed by inspecting the function's between-peak distance inputs
- The output feature matrix from BuildFeatureMatrix on regrouped peaks has the same samples and consistent group structure as before regrouping

## Limitations

- SilhouetR is described as an optional detection step; grouping errors that do not manifest as low mean silhouette values may go undetected
- The wine-data diagnostic threshold (mean silhouette ~0.25 with high average SNR) is dataset-specific; other datasets may need recalibrated criteria
- regroupR relies on ppm and SNR distances only; peaks with ambiguous ppm/SNR signatures may not be reassigned to the intended group

## Evidence

- [supplementary] SilhouetR is optional but recommended for detection: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [supplementary] Wine data as validation dataset: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [abstract] Peak picking and grouping workflow context: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [abstract] speaq distributed as R package: "speaq 2.0 is distributed through the existing speaq R package"
- [readme] Peak grouping invocation on wine data: [evidence span withheld — anchor status: not_found]
