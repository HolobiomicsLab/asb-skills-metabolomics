---
name: nmr-feature-differential-analysis
description: Use when you have a speaq 2.0 peak-picked/grouped feature matrix (samples
  x features) plus class metadata for 1D NMR spectra, and you need to find features
  that significantly differ between two classes (e.g. red vs white wines).
when_to_use_negative:
- Input is already a summarized/binned feature table not produced by speaq 2.0's peak-picking-and-grouping
  pipeline (the method avoids binning, so binned inputs change feature semantics)
- Raw unprocessed spectra without prior peak picking, grouping, and feature-matrix
  construction — run getWaveletPeaks/PeakGrouper/BuildFeatureMatrix first
- Analysis goal requiring multivariate modelling (e.g. PCA, supervised classification)
  rather than per-feature univariate class comparison
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: speaq 2.0
  role: Provides the differential analysis routine (linear modeling per feature with
    Bonferroni correction) applied to the Pareto-scaled feature matrix
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R distribution of speaq 2.0; supplies getWaveletPeaks, PeakGrouper, BuildFeatureMatrix,
    scaling/imputation, and differential analysis functions
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq pipeline and linear-model differential
    analysis are executed
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:827ef2ffdae8e204f4fd90dc736794b5a80c5b1973198d406446d8d0ccefc268
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nmr-feature-differential-analysis

## Summary

Univariate differential analysis of a speaq 2.0-derived NMR peak-based feature matrix (e.g. red vs white wine classes) using linear models per feature with Bonferroni correction, as implemented in the speaq R package. It identifies the spectral features that significantly discriminate predefined sample classes after Pareto scaling.

## When to use

Use when you have a speaq 2.0 peak-picked/grouped feature matrix (samples x features) plus class metadata for 1D NMR spectra, and you need to find features that significantly differ between two classes (e.g. red vs white wines).

## When NOT to use

- Input is already a summarized/binned feature table not produced by speaq 2.0's peak-picking-and-grouping pipeline (the method avoids binning, so binned inputs change feature semantics)
- Raw unprocessed spectra without prior peak picking, grouping, and feature-matrix construction — run getWaveletPeaks/PeakGrouper/BuildFeatureMatrix first
- Analysis goal requiring multivariate modelling (e.g. PCA, supervised classification) rather than per-feature univariate class comparison

## Inputs

- speaq 2.0 NMR feature matrix (samples x peak-group features) from BuildFeatureMatrix, e.g. Winedata$spectra processed spectra
- Class metadata factor per sample (e.g. Winedata$wine.color: red, white, rosé)
- ppm vector aligned with spectra columns (Winedata$ppm)

## Outputs

- wine_differential_analysis_results.csv with one row per tested feature: feature, coefficient, raw p-value, Bonferroni-adjusted p-value, significance flag
- Set of significant discriminating features (33 for red vs white wine) with adjusted p-values below the family-wise threshold

## How to apply

Build the feature matrix with speaq 2.0's pipeline: getWaveletPeaks (wavelet peak picking), PeakGrouper (grouping), and BuildFeatureMatrix, avoiding the binning step. Remove samples not belonging to the two classes of interest (e.g. drop the 2 rosé wines so only red vs white remain). Apply Pareto scaling to the feature matrix columns as the standard metabolomics scaling prior to univariate statistics. For each feature, fit a linear model of feature intensity on the wine class factor using speaq 2.0's differential analysis routine (which supports more than 2 groups), then apply Bonferroni correction to the per-feature p-values and select features surviving the family-wise threshold (e.g. 33 significant features in the wine red-vs-white analysis). Write the full results table (feature, coefficient, raw and Bonferroni-adjusted p-values, significance flag) with one row per tested feature.

## Related tools

- **speaq 2.0** (Provides the differential analysis routine (linear modeling per feature with Bonferroni correction) applied to the Pareto-scaled feature matrix) — https://github.com/beirnaert/speaq
- **speaq R package** (R distribution of speaq 2.0; supplies getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, scaling/imputation, and differential analysis functions) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq pipeline and linear-model differential analysis are executed) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec=as.matrix(Winedata$spectra), X.ppm=as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks=peaks); Features <- BuildFeatureMatrix(Y.data=groups)
```

## Evaluation signals

- Corrected analysis on the red-vs-white wine feature matrix yields exactly the reported 33 significant features separating the two classes
- Results file contains one row per tested feature and a valid Bonferroni-adjusted p-value column
- Rosé samples are excluded so only red vs white classes enter the linear models
- Feature matrix columns are Pareto-scaled before fitting the per-feature linear models
- Significant features show class-effect coefficients consistent with the PCA separation of red and white wines described in the article's supplementary wine data analysis

## Limitations

- The differential analysis is univariate per feature; no multivariate significance assessment is provided
- Peak grouping quality affects features — speaq 2.0 can delete a large peak in the grouping step and peak filling does not always mitigate it; the optional SilhouetR function detects such problems via silhouette values per group
- Spectral alignment of severely shifted spectra is not always optimal, which can fragment features before testing

## Evidence

- [abstract] Feature/sample/peak-value matrix via wavelet peak picking and grouping: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README minimal example on Winedata with wine.color class factor: [evidence span withheld — anchor status: not_found]
- [readme] README lists differential analysis among peak-based features: [evidence span withheld — anchor status: external_doc]
- [supplementary] Wine dataset class separation in supplementary PCA: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [supplementary] SilhouetR detection of grouping problems: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
