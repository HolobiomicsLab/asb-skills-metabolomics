---
name: pca-scores-interpretation-metabolomics
description: Use when you have a peak-value feature matrix from 1D NMR metabolomics
  spectra and need to visually assess whether experimental groups (e.g., diet concentrations)
  separate before or alongside differential analysis.
when_to_use_negative:
- The input is already a reduced/dimensionality-transformed table (e.g., PCA scores
  themselves) rather than a raw peak-value feature matrix
- You need quantitative feature-level significance calls with multiple-testing correction
  — use a Bonferroni-corrected linear model on the same features instead of relying
  on PCA trends
- Spectra are severely shifted and no peak-based feature matrix has been built yet;
  spectral alignment issues must be resolved upstream first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: speaq 2.0
  role: Builds the peak-value feature matrix (peak picking, grouping, peak filling)
    that PCA is run on; provides statistical/visualization tooling for the feature
    matrix
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R implementation through which the feature matrix and PCA analysis are executed
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment for running speaq functions and PCA on the feature matrix
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:6e3f7b405040e8ab3d458a3bc23ecaca1e001eb03a56d0b5f5edb8ee94975e2f
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pca-scores-interpretation-metabolomics

## Summary

Run and interpret PCA on a peak-based feature matrix from 1D NMR spectra (e.g., the 31 samples x 677 features matrix produced by the speaq 2.0 workflow) to check whether experimental groups show separation trends in the score plot.

## When to use

Use when you have a peak-value feature matrix from 1D NMR metabolomics spectra and need to visually assess whether experimental groups (e.g., diet concentrations) separate before or alongside differential analysis.

## When NOT to use

- The input is already a reduced/dimensionality-transformed table (e.g., PCA scores themselves) rather than a raw peak-value feature matrix
- You need quantitative feature-level significance calls with multiple-testing correction — use a Bonferroni-corrected linear model on the same features instead of relying on PCA trends
- Spectra are severely shifted and no peak-based feature matrix has been built yet; spectral alignment issues must be resolved upstream first

## Inputs

- Sample-by-feature peak-value matrix (e.g., 31 samples x 677 features from speaq 2.0 peak picking, grouping, and peak filling on 1D NMR spectra)
- Group/class factor per sample (e.g., onion diet percentage as a factor)

## Outputs

- PCA score plot with samples colored by group factor
- Qualitative assessment of group separation trends (e.g., 'no consistent separation by diet group')
- Optionally the feature matrix and differential-analysis results written to a CSV such as speaq_onion_results.csv alongside the PCA assessment

## How to apply

Build the feature matrix first (speaq 2.0: Mexican hat wavelet peak picking, peak grouping, and peak filling yield the 31 x 677 peak-value matrix for the onion intake in mice urine data). Apply scaling/transformations to the feature matrix if needed, then run PCA on the samples-by-features matrix. Project the samples onto the first principal components and color the score plot by the group factor (e.g., increasing onion diet percentage). Interpret separation cautiously: in the onion intake study the PCA score plot showed no consistent group separation by diet group even though the Bonferroni-corrected linear model on the same features identified 9 significant peaks, so a null PCA trend does not invalidate significant univariate features. The README shows the same pattern on the wine data, where PC2 clearly separated red, white and rosé wines.

## Related tools

- **speaq 2.0** (Builds the peak-value feature matrix (peak picking, grouping, peak filling) that PCA is run on; provides statistical/visualization tooling for the feature matrix) — https://github.com/beirnaert/speaq
- **speaq R package** (R implementation through which the feature matrix and PCA analysis are executed) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment for running speaq functions and PCA on the feature matrix) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec=spectra.matrix, X.ppm=ppm.vector); Features <- BuildFeatureMatrix(Y.data=PeakGrouper(Y.peaks=peaks)); prcomp(Features)
```

## Evaluation signals

- PCA is computed on the full 31 x 677 peak-value feature matrix, not on a subset or binned table
- Score plot axes are principal components and points are colored by the group factor (e.g., diet percentage)
- Interpretation is consistent with the expected result: the onion intake in mice data show no consistent group separation by diet group in the PCA score plot
- A null PCA separation trend is not treated as contradicting the 9 Bonferroni-significant peaks found by the linear model on the same features
- Positive control: on the wine dataset the score plot should show PC2 separating red, white and rosé wines

## Limitations

- PCA separation in the score plot is a qualitative check; absence of visible group separation (as for the onion diet groups) does not preclude significant features under Bonferroni-corrected differential analysis
- Result quality depends on the upstream peak grouping and peak filling steps; artifacts such as deleted large peaks in grouping can propagate into the feature matrix
- Severely shifted spectra can degrade feature matrix construction, affecting PCA patterns

## Evidence

- [supplementary] PCA on feature matrix with group-color check: "speaq 2.0 workflow applied to the onion intake in mice data [37]."
- [supplementary] wine data PCA separation positive control: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [intro] statistical analysis follows pre-processing: "consists of several pre-processing steps, such as baseline correction, raw spectral alignment, spectra summarization and grouping. This is then followed by statistical analysis."
- [readme] README minimal example: feature matrix then PCA: "this featurematrix can be processed further (scaling, transformations) or 
# analysed with the statistical tools of interest like PCA."
- [abstract] workflow avoids binning, yields feature matrix: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
