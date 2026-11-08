---
name: peak-filling-gap-imputation
description: Use when a speaq 2.0 peak-based workflow has produced grouped peak lists
  from 1D NMR spectra and some spectra lack a peak in groups present in other spectra,
  requiring gap filling before building the feature matrix.
when_to_use_negative:
- Input is already a completed feature matrix — filling operates on grouped peak lists,
  not on the final matrix
- Raw spectra still need peak detection or grouping; peak filling assumes wavelet
  peak picking (getWaveletPeaks) and grouping (PeakGrouper) have already run
- The analysis requires the alignment-based speaq v1.2.3 workflow (CluPA spectral
  alignment, BW-ratio quantitation) instead of peak-based processing
edam_operation: http://edamontology.org/operation_3557
tools:
- name: speaq 2.0
  role: Provides the peak filling step within its peak-picking -> grouping -> peak
    filling workflow
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributed implementation; supplies BuildFeatureMatrix to assemble the filled
    feature matrix
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq package functions (PeakGrouper, BuildFeatureMatrix)
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:4abcbb7d19b7e07b1fcffd38e72ac982e16cd5e913d76b2acbcac122104eab99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3557
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# peak-filling-gap-imputation

## Summary

The speaq 2.0 peak filling step recovers peak values in groups where the wavelet-based peak picker missed a peak in one or more of the 40 wine NMR spectra, completing the 40-sample by 207-peak-group feature matrix without binning.

## When to use

Use when a speaq 2.0 peak-based workflow has produced grouped peak lists from 1D NMR spectra and some spectra lack a peak in groups present in other spectra, requiring gap filling before building the feature matrix.

## When NOT to use

- Input is already a completed feature matrix — filling operates on grouped peak lists, not on the final matrix
- Raw spectra still need peak detection or grouping; peak filling assumes wavelet peak picking (getWaveletPeaks) and grouping (PeakGrouper) have already run
- The analysis requires the alignment-based speaq v1.2.3 workflow (CluPA spectral alignment, BW-ratio quantitation) instead of peak-based processing

## Inputs

- Grouped peak lists from speaq 2.0 PeakGrouper (per-spectrum wavelet-detected peaks grouped across 40 wine spectra)
- Raw 1D NMR spectra matrix (e.g. Winedata$spectra, 40 spectra x 8712 measurement points) with ppm vector

## Outputs

- Completed feature/sample/peak-value matrix (40 samples x 207 peak groups for the wine data), e.g. written to feature_matrix.csv

## How to apply

After detectSpecPeaks/getWaveletPeaks performs Mexican hat continuous wavelet peak detection on each 1D NMR spectrum (e.g. the 40 table wine spectra with 8712 measurement points) and PeakGrouper groups detected peaks across spectra into peak groups, run the peak filling step to impute peak values in groups where a peak was missed. The rationale is that peak pickers occasionally miss genuine peaks in individual spectra; without filling, those cells of the feature/sample/peak-value matrix would be empty and the data reduction would be inconsistent across samples. Filling feeds directly into BuildFeatureMatrix, which assembles the completed matrix (for the wine data: 40 samples x 207 peak groups). Optionally use SilhouetR to compute silhouette values per group to detect problematic groups that filling could not mitigate. The resulting matrix can then be scaled, transformed, or analysed (e.g. PCA separating red, white and rose wines).

## Related tools

- **speaq 2.0** (Provides the peak filling step within its peak-picking -> grouping -> peak filling workflow) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributed implementation; supplies BuildFeatureMatrix to assemble the filled feature matrix) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq package functions (PeakGrouper, BuildFeatureMatrix) are executed)

## Examples

```
Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Resulting feature matrix has 207 peak groups (features) by 40 samples when applied to the wine dataset (originally 40 x 8712)
- No empty/missing peak-value cells remain in the matrix after filling followed by BuildFeatureMatrix
- Each peak group retains values for all 40 wine spectra, including spectra where peak picking initially missed a peak
- Optional SilhouetR silhouette values per group do not flag problematic groups that filling failed to mitigate
- Downstream PCA of the matrix reproduces the expected separation of red, white and rose wines on Principal Component 2

## Limitations

- Peak filling does not always mitigate missed peaks: 'This problem is usually mitigated by the peak filling step but in this case it is not' — some problematic groups remain after filling
- Severely shifted spectra can degrade upstream grouping so that filling cannot recover a coherent group value
- The grouping step may delete one of the large peaks, and SilhouetR only detects (does not fix) such problems

## Evidence

- [supplementary] This problem is usually mitigated by the peak filling step but in this case it is not.: "This problem is usually mitigated by the peak filling step but in this case it is not."
- [abstract] Workflow avoids binning and yields a feature matrix: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README minimal example assembles the feature matrix after grouping: "# get the feature matrix
Features <- BuildFeatureMatrix(Y.data = groups)"
- [readme] README lists peak filling among the peak-based approach features: "* Peak based approach (new since v2.0)
	- peak picking (wavelets)
	- grouping
	- peak filling"
- [other] Source card finding on wine data dimensions: [evidence span withheld — anchor status: not_found]
