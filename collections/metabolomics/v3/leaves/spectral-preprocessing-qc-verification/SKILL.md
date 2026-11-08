---
name: spectral-preprocessing-qc-verification
description: Use when a raw 1D NMR spectra matrix (samples x ppm points, e.g. 40 wines
  x 8712 points) has been reduced by peak-picking/grouping/peak filling and the output
  feature matrix must be validated before downstream statistics.
when_to_use_negative:
- Input is already a binned/bucketed feature table or a feature/sample/peak-value
  matrix produced by a prior run — QC applies to the raw spectra-to-features reduction
- The spectra are severely shifted and only an alignment method (CluPA, speaq <= 1.2.3)
  is needed rather than peak-based reduction QC
- Data are not 1D NMR spectra (e.g. LC-MS feature tables), where XCMS-style workflows
  rather than speaq 2.0 apply
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: speaq 2.0
  role: Provides the wavelet peak-picking, grouping, peak filling and SilhouetR QC
    functions being verified
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN distribution through which speaq 2.0 functions (getWaveletPeaks, PeakGrouper,
    BuildFeatureMatrix) are invoked in R
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment for executing and checking the workflow outputs
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:2c4ba78ce1fb8eb2706f4983ba22bb0d5ec2bfc9e8eb1e13d4eb3cb9658bc4b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-preprocessing-qc-verification

## Summary

Quality-control the speaq 2.0 peak-based NMR preprocessing pipeline (wavelet peak picking, grouping, peak filling) by verifying that the resulting feature/sample/peak-value matrix has the expected dimensions and grouping integrity, e.g. 40 wine spectra by 207 peak groups.

## When to use

Use when a raw 1D NMR spectra matrix (samples x ppm points, e.g. 40 wines x 8712 points) has been reduced by peak-picking/grouping/peak filling and the output feature matrix must be validated before downstream statistics.

## When NOT to use

- Input is already a binned/bucketed feature table or a feature/sample/peak-value matrix produced by a prior run — QC applies to the raw spectra-to-features reduction
- The spectra are severely shifted and only an alignment method (CluPA, speaq <= 1.2.3) is needed rather than peak-based reduction QC
- Data are not 1D NMR spectra (e.g. LC-MS feature tables), where XCMS-style workflows rather than speaq 2.0 apply

## Inputs

- Raw 1D NMR spectra matrix (40 wine samples x 8712 ppm measurement points)
- ppm vector for the spectra matrix
- Class factor / sample grouping (e.g. wine color: red, white, rosé)

## Outputs

- Feature matrix of 40 samples by 207 peak groups (feature_matrix.csv)
- Per-peak-group silhouette values from SilhouetR for grouping QC
- PCA score plot as a sanity check on class separation

## How to apply

Run the speaq 2.0 workflow end to end: wavelet-based peak detection on each spectrum (getWaveletPeaks, Mexican hat CWT), cross-spectral peak grouping (PeakGrouper), and peak filling to impute missed peaks, then build the feature matrix with BuildFeatureMatrix. Verify the output dimensions against the known reduction — for the wine dataset (40 table wines, 8712 measurement points) the matrix must be 40 samples by 207 peak-group features — and export it (e.g. feature_matrix.csv). As an additional grouping QC, run the optional SilhouetR function, which calculates silhouette values per peak group to detect problematic groups (e.g. a deleted large peak that peak filling fails to mitigate). Finally sanity-check the reduced data biologically, e.g. by PCA where Principal Component 2 should separate red, white and rosé wines.

## Related tools

- **speaq 2.0** (Provides the wavelet peak-picking, grouping, peak filling and SilhouetR QC functions being verified) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN distribution through which speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) are invoked in R) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment for executing and checking the workflow outputs) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups); dim(Features)  # expect 40 x 207
```

## Evaluation signals

- Feature matrix dimensions equal the expected reduction: 40 samples x 207 peak groups for the wine dataset (from 40 x 8712 raw points)
- Peak filling has imputed peak values in all groups where detection was missed, so no peak group lacks values for any of the 40 samples
- SilhouetR silhouette values per peak group show no anomalous group (e.g. a group where a large peak was deleted)
- PCA of the feature matrix separates red, white and rosé wines along Principal Component 2
- feature_matrix.csv is written with the sample-by-feature schema matching the speaq 2.0 output

## Limitations

- The peak grouping step may delete one of the large peaks, and the peak filling step does not always mitigate this problem
- Alignment-based processing (relevant to speaq <= 1.2.3) is not always optimal for severely shifted spectra
- speaq 2.0 aims at minimal but not zero information loss: wavelet summarization of raw spectra retains only detected peaks, unlike raw-bin representations

## Evidence

- [abstract] wavelet-based peak picking on each spectrum: "uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction"
- [abstract] workflow order of peak picking then grouping without binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] SilhouetR as optional grouping QC: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [supplementary] peak filling limitation: "This problem is usually mitigated by the peak filling step but in this case it is not."
- [supplementary] wine PCA sanity check: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [readme] R invocation of the pipeline stages to verify: [evidence span withheld — anchor status: not_found]
