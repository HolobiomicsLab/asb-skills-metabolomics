---
name: pareto-scaling-metabolomics-matrix
description: Use when a peak-based feature matrix has been built with speaq 2.0 (BuildFeatureMatrix)
  and a univariate test such as red-vs-white linear-model differential analysis requires
  column scaling as a pre-statistics step.
when_to_use_negative:
- The input is already a scaled feature table or you plan a method that handles scaling
  internally — re-scaling can double-transform the data.
- Raw 1D NMR spectra rather than a peak-picked/grouped feature matrix — run the speaq
  2.0 peak picking and grouping workflow first.
- Binned/bucketed data from other NMR frameworks — speaq 2.0's paradigm avoids the
  binning step, and its scaling is designed for the peak-value matrix it produces.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: speaq 2.0
  role: provides the peak-based workflow (peak picking, grouping, BuildFeatureMatrix)
    whose output feature matrix is Pareto-scaled, plus 'scaling & imputations' and
    'differential analysis (also more than 2 groups)' functions
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R distribution of speaq 2.0; installed from CRAN and used to run BuildFeatureMatrix
    and the scaling/statistical functions
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: runtime environment in which the speaq R package functions are called
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:09c32af46ee2227924f5866375ae263238cb6463a3cfdd4df6092983566759d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pareto-scaling-metabolomics-matrix

## Summary

Apply Pareto scaling to a speaq 2.0-derived NMR feature/sample/peak-value matrix (e.g. the wine NMR feature matrix) so that downstream univariate statistics such as linear-model differential analysis compare features on a common scale while partially preserving large peaks.

## When to use

Use when a peak-based feature matrix has been built with speaq 2.0 (BuildFeatureMatrix) and a univariate test such as red-vs-white linear-model differential analysis requires column scaling as a pre-statistics step.

## When NOT to use

- The input is already a scaled feature table or you plan a method that handles scaling internally — re-scaling can double-transform the data.
- Raw 1D NMR spectra rather than a peak-picked/grouped feature matrix — run the speaq 2.0 peak picking and grouping workflow first.
- Binned/bucketed data from other NMR frameworks — speaq 2.0's paradigm avoids the binning step, and its scaling is designed for the peak-value matrix it produces.

## Inputs

- speaq 2.0 peak-based feature matrix (from getWaveletPeaks, PeakGrouper and BuildFeatureMatrix)
- class metadata factor (e.g. Winedata$wine.color: red, white, rosé)
- optional: ppm vector and spectra matrix from Winedata

## Outputs

- Pareto-scaled feature matrix (same feature rows and sample columns, columns scaled to unit-variance/sqrt(sd))
- downstream: differential analysis results table (feature, coefficient, raw and Bonferroni-adjusted p-values, significance flag), e.g. wine_differential_analysis_results.csv

## How to apply

Build the feature matrix from wavelet-picked, grouped peaks using speaq 2.0's getWaveletPeaks, PeakGrouper and BuildFeatureMatrix, as shown in the package README. The README notes the resulting feature matrix 'can be processed further (scaling, transformations)' before statistical analysis, and the peak-based feature set explicitly includes 'scaling & imputations'. Apply Pareto scaling to the feature matrix columns — the standard metabolomics scaling that divides each feature by the square root of its standard deviation — prior to univariate statistics. In the wine NMR use case, first remove non-target classes (e.g. the 2 rosé wines) so only red vs white remain, Pareto-scale the matrix, then feed it into speaq 2.0's differential analysis (linear model of feature intensity on wine class, Bonferroni-corrected), which reproduces 33 significant discriminating features.

## Related tools

- **speaq 2.0** (provides the peak-based workflow (peak picking, grouping, BuildFeatureMatrix) whose output feature matrix is Pareto-scaled, plus 'scaling & imputations' and 'differential analysis (also more than 2 groups)' functions) — https://github.com/beirnaert/speaq
- **speaq R package** (R distribution of speaq 2.0; installed from CRAN and used to run BuildFeatureMatrix and the scaling/statistical functions) — https://cran.r-project.org/package=speaq
- **R** (runtime environment in which the speaq R package functions are called) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); Features <- BuildFeatureMatrix(Y.data = PeakGrouper(Y.peaks = getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)))); Features.scaled <- ScaleFeatures(Features.scaled = Features, method = "Pareto")
```

## Evaluation signals

- Scaling is applied per column of the feature matrix; after Pareto scaling each column's standard deviation equals 1 only in the variance sense (sd scaled by sqrt(sd)), so check column variances equal their original standard deviations.
- The scaled matrix must retain the same dimensions (features x samples) and row/column names as the BuildFeatureMatrix output.
- Downstream red-vs-white differential analysis on the Pareto-scaled matrix must yield exactly 33 significant features after Bonferroni correction.
- The results file must contain one row per tested feature with a valid Bonferroni-adjusted p-value column.
- A PCA on the scaled wine features should reproduce the article's observation that Principal Component 2 separates red, white and rosé wines.

## Limitations

- Scaling operates on the peak-value feature matrix, not raw spectra; severe spectral shifts can degrade the underlying peak grouping and hence the feature values being scaled.
- Class imbalance or non-target classes (e.g. rosé wines in the wine dataset) should be handled before computing scaling statistics for a red-vs-white comparison.
- speaq 2.0's peak filling step may not always mitigate missing peaks, which can leave gaps in the feature matrix prior to scaling and imputation.

## Evidence

- [readme] README scaling of feature matrix: "this featurematrix can be processed further (scaling, transformations) or 
# analysed with the statistical tools of interest like PCA."
- [readme] README peak-based features include scaling: "peak picking (wavelets)
	- grouping
	- peak filling
	- scaling & imputations
	- differential analysis (also more than 2 groups)"
- [abstract] speaq 2.0 avoids binning, produces feature matrix: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] Wine PCA validation of classes: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
