---
name: wine-metabolomics-class-discrimination
description: Use when a peak-based NMR feature/sample/peak-value matrix (e.g. from
  speaq 2.0 peak picking, grouping and BuildFeatureMatrix) must be tested for features
  that significantly differ between two or more sample classes.
when_to_use_negative:
- Input is raw 1D spectra rather than a peak-grouped feature matrix — peak picking,
  grouping and peak filling (getWaveletPeaks/PeakGrouper/BuildFeatureMatrix) must
  run first.
- The analysis goal is spectral alignment or quantitation of raw spectra (CluPA alignment,
  BW-ratio quantitation), which are separate speaq workflows for versions <= 1.2.3
  features.
- Class labels are unknown or the goal is unsupervised exploration — use PCA as in
  the paper's wine data supplementary analysis instead of the class-based linear model.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: speaq 2.0
  role: Provides the differential analysis routine (linear modeling with p-value correction)
    applied to the Pareto-scaled wine feature matrix
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributes speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix,
    differential analysis) installable from CRAN
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq package linear-model differential analysis
    and Bonferroni correction are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:10a19b0aa05eee99dad720531d5d71cc9b84fe38711669760d5ddd3fd38a3c71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wine-metabolomics-class-discrimination

## Summary

Apply speaq 2.0's linear-model differential analysis to a Pareto-scaled NMR feature matrix (e.g. the wine spectra dataset) to identify features that significantly discriminate between predefined sample classes (red vs white wines).

## When to use

Use when a peak-based NMR feature/sample/peak-value matrix (e.g. from speaq 2.0 peak picking, grouping and BuildFeatureMatrix) must be tested for features that significantly differ between two or more sample classes.

## When NOT to use

- Input is raw 1D spectra rather than a peak-grouped feature matrix — peak picking, grouping and peak filling (getWaveletPeaks/PeakGrouper/BuildFeatureMatrix) must run first.
- The analysis goal is spectral alignment or quantitation of raw spectra (CluPA alignment, BW-ratio quantitation), which are separate speaq workflows for versions <= 1.2.3 features.
- Class labels are unknown or the goal is unsupervised exploration — use PCA as in the paper's wine data supplementary analysis instead of the class-based linear model.

## Inputs

- speaq-derived NMR feature matrix (features x samples peak-value matrix from BuildFeatureMatrix)
- Class metadata factor (e.g. Winedata$wine.color with red/white/rosé levels)
- ppm vector for feature annotation (Winedata$ppm)

## Outputs

- List/count of significant discriminating features (33 for the wine red-vs-white contrast)
- Full results table per feature: coefficient, raw p-value, Bonferroni-adjusted p-value, significance flag (written to wine_differential_analysis_results.csv)

## How to apply

Start from the speaq 2.0 feature matrix (getWaveletPeaks → PeakGrouper → BuildFeatureMatrix) plus a class factor (e.g. Winedata$wine.color); remove classes outside the contrast of interest (drop the 2 rosé wines to leave red vs white). Apply Pareto scaling to the feature columns as the standard metabolomics scaling before univariate statistics. For each feature, fit a linear model of feature intensity on wine class using speaq 2.0's differential analysis routine built on R's linear modeling (the package supports more than 2 groups). Apply Bonferroni correction to per-feature p-values and select features surviving the family-wise threshold; for the wine data this selection is expected to yield exactly 33 significant features separating red and white wines.

## Related tools

- **speaq 2.0** (Provides the differential analysis routine (linear modeling with p-value correction) applied to the Pareto-scaled wine feature matrix) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributes speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, differential analysis) installable from CRAN) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq package linear-model differential analysis and Bonferroni correction are executed)

## Examples

```
library(speaq); F <- BuildFeatureMatrix(PeakGrouper(getWaveletPeaks(Winedata$spectra, Winedata$ppm)))  # then Pareto-scale and run speaq differential analysis on wine.color
```

## Evaluation signals

- The Bonferroni-corrected analysis must yield exactly the reported 33 significant features separating red and white wines.
- The results file (wine_differential_analysis_results.csv) must contain one row per tested feature with a valid Bonferroni-adjusted p-value column.
- Only red and white classes remain in the model fit — the 2 rosé wines must have been removed before testing.
- All feature columns must be Pareto-scaled prior to the univariate linear models.
- Every feature's adjusted p-value must be non-decreasing relative to its raw p-value and bounded in [0,1].

## Limitations

- The expected 33 significant features is specific to the wine dataset and red-vs-white contrast; other datasets will have different counts.
- The optional SilhouetR function exists because grouping errors can delete or misassign large peaks, corrupting the feature matrix upstream of the statistics ('The speaq 2.0 algorithm deletes one of the large peaks in the grouping step'; peak filling 'in this case it is not' sufficient).
- Peak filling may not mitigate all missing-peak problems, so some features may be absent from the matrix before analysis.

## Evidence

- [readme] workflow step: differential analysis after scaling: [evidence span withheld — anchor status: not_found]
- [readme] feature matrix construction functions: [evidence span withheld — anchor status: not_found]
- [abstract] workflow avoids binning, produces feature matrix: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] wine dataset classes red/white/rosé: "Principal Component 2 clearly indicates a difference between red, white and rosé wines."
