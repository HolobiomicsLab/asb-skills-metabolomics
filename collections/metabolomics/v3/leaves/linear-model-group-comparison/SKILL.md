---
name: linear-model-group-comparison
description: Use when you have a peak-based feature/sample matrix from speaq 2.0 (e.g.
  from BuildFeatureMatrix on Winedata) plus class metadata and need a per-feature
  significance test between two groups.
when_to_use_negative:
- The input is already a list of significant features or a finished differential-analysis
  results table rather than a feature matrix to be tested
- The comparison involves more than the two designated groups without adjusting the
  linear model design (though speaq 2.0's differential analysis does also support
  more than 2 groups)
- The data are raw 1D spectra that have not been peak-picked, grouped and filled into
  a feature matrix — run the speaq 2.0 peak-based workflow first
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides the differential analysis routine (linear modeling with Bonferroni
    correction) and the upstream wavelet peak-picking, grouping and feature-matrix
    construction applied to 1D NMR spectra
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R distribution of speaq 2.0 through which the statistical functions are invoked
    (install.packages("speaq"))
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the linear models, Pareto scaling and Bonferroni
    correction are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:596498c9fe51a8b31f75a58c8c680b579f1efef64dfd4de9b321b3ac838ac3df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# linear-model-group-comparison

## Summary

Per-feature univariate differential analysis of a speaq 2.0-derived NMR feature matrix using linear models on a class factor, with Bonferroni correction of p-values, to identify features that significantly discriminate between groups (e.g. 33 significant red-vs-white wine features).

## When to use

Use when you have a peak-based feature/sample matrix from speaq 2.0 (e.g. from BuildFeatureMatrix on Winedata) plus class metadata and need a per-feature significance test between two groups.

## When NOT to use

- The input is already a list of significant features or a finished differential-analysis results table rather than a feature matrix to be tested
- The comparison involves more than the two designated groups without adjusting the linear model design (though speaq 2.0's differential analysis does also support more than 2 groups)
- The data are raw 1D spectra that have not been peak-picked, grouped and filled into a feature matrix — run the speaq 2.0 peak-based workflow first

## Inputs

- speaq 2.0 feature matrix (features x samples) from BuildFeatureMatrix on grouped wavelet peaks
- Sample class metadata as a factor (e.g. Winedata$wine.color)
- PPM vector and raw spectra matrix for upstream peak detection (Winedata$spectra, Winedata$ppm)

## Outputs

- Results table with one row per tested feature: feature, coefficient, raw p-value, Bonferroni-adjusted p-value, significance flag (e.g. wine_differential_analysis_results.csv)
- Count of significant features discriminating the classes (e.g. 33 significant red-vs-white features)

## How to apply

Start from the speaq 2.0 feature matrix produced by getWaveletPeaks, PeakGrouper and BuildFeatureMatrix, and the class factor (e.g. wine.color). Remove samples outside the two groups of interest (e.g. drop the 2 rosé wines to compare red vs white only), then apply Pareto scaling to the feature matrix columns as the standard metabolomics scaling before univariate statistics. For each feature, fit a linear model of feature intensity on the class factor using speaq 2.0's differential analysis routine built on R's linear modeling. Apply Bonferroni correction to the per-feature p-values and select features surviving the family-wise significance threshold. Write the full results table (feature, coefficient, raw and corrected p-values, significance flag) so the selection can be validated.

## Related tools

- **speaq 2.0** (Provides the differential analysis routine (linear modeling with Bonferroni correction) and the upstream wavelet peak-picking, grouping and feature-matrix construction applied to 1D NMR spectra) — https://github.com/beirnaert/speaq
- **speaq R package** (R distribution of speaq 2.0 through which the statistical functions are invoked (install.packages("speaq"))) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the linear models, Pareto scaling and Bonferroni correction are computed)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- On the wine dataset (rosé wines removed), the Bonferroni-corrected linear-model analysis must yield exactly the reported 33 significant features separating red and white wines
- The results file contains one row per tested feature, and every row has a valid Bonferroni-adjusted p-value
- The feature matrix columns were Pareto scaled before fitting the linear models
- Only red and white samples are in the model: the 2 rosé wines are removed from the class factor prior to testing

## Limitations

- The grouping/peak-filling stage of speaq 2.0 can delete or fail to fill large peaks (e.g. in severely shifted spectra), so some discriminating features may be missing from the feature matrix before testing
- Univariate linear models treat features independently; the paper's supplementary PCA shows classes separate on principal components, so correlated feature structure is not modeled

## Evidence

- [supplementary] Wine data PCA plot.The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines.: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README differential analysis feature: [evidence span withheld — anchor status: external_doc]
- [readme] README feature matrix usage: "# this featurematrix can be processed further (scaling, transformations) or 
# analysed with the statistical tools of interest like PCA."
