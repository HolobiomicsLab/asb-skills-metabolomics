---
name: bonferroni-multiple-testing-correction
description: Use when a univariate linear model is fitted on each column of an NMR
  peak-value feature matrix (e.g., 31 x 677 from speaq 2.0) and raw p-values must
  be family-wise corrected before declaring significant features.
when_to_use_negative:
- The input is raw 1D NMR spectra rather than a completed peak-value feature matrix
  — peak picking, grouping, and peak filling must be run first (getWaveletPeaks, PeakGrouper,
  BuildFeatureMatrix).
- The feature matrix columns are unscaled — apply the metabolomics scaling (e.g.,
  Pareto scaling) before univariate statistics, as done in the wine analysis.
- Only a handful of hypothesis tests are involved, where a family-wise Bonferroni
  correction across hundreds of features is not the intended multiplicity scope.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: speaq 2.0
  role: Provides the differential analysis routine whose per-feature linear-model
    p-values are Bonferroni-corrected
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: CRAN distribution used in R to load spectra, build the feature matrix, and
    run scaling, imputation, and differential analysis
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the linear models and Bonferroni adjustment are
    computed
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:f4b32152c00a5ecba38672ba0d8c34ce571f97a88efb58c78beb5d65a21dfb11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bonferroni-multiple-testing-correction

## Summary

Bonferroni family-wise correction applied to per-feature p-values from speaq 2.0's linear-model differential analysis on an NMR peak-value feature matrix, used to call significant discriminating peaks (e.g., 33 red-vs-white wine features; 9 peaks in the onion intake in mice study).

## When to use

Use when a univariate linear model is fitted on each column of an NMR peak-value feature matrix (e.g., 31 x 677 from speaq 2.0) and raw p-values must be family-wise corrected before declaring significant features.

## When NOT to use

- The input is raw 1D NMR spectra rather than a completed peak-value feature matrix — peak picking, grouping, and peak filling must be run first (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix).
- The feature matrix columns are unscaled — apply the metabolomics scaling (e.g., Pareto scaling) before univariate statistics, as done in the wine analysis.
- Only a handful of hypothesis tests are involved, where a family-wise Bonferroni correction across hundreds of features is not the intended multiplicity scope.

## Inputs

- speaq 2.0 peak-value feature matrix (e.g., 31 samples x 677 features for the onion intake in mice urine NMR data)
- class or dose metadata vector per sample (e.g., wine color factor with rosé removed, or increasing onion percentage)
- feature-wise linear-model p-values from speaq 2.0's differential analysis routine

## Outputs

- Bonferroni-adjusted p-value vector, one entry per tested feature
- Significance flag per feature surviving the family-wise threshold (e.g., 33 significant red-vs-white wine features; 9 significant onion-diet peaks)
- Results table (feature, coefficient, raw and corrected p-values, significance flag) such as wine_differential_analysis_results.csv or speaq_onion_results.csv

## How to apply

First build the speaq 2.0 feature matrix via peak detection (Mexican hat wavelet), peak grouping, and peak filling, then scale the feature columns (e.g., Pareto scaling, as in the red-vs-white wine analysis) as standard metabolomics pre-treatment. For every feature, fit a linear model of feature intensity on the class or dose variable (wine color, or increasing onion diet percentage) using speaq 2.0's differential analysis built on R's linear modeling. Collect the per-feature raw p-values and multiply each by the total number of tested features (Bonferroni; 677 tests for the onion matrix), then select features whose adjusted p-value passes the family-wise significance threshold. Write out a results table with one row per tested feature containing feature, coefficient, raw p-value, Bonferroni-adjusted p-value, and significance flag. The corrected feature lists feed downstream interpretation such as querying HMDB with peak ppm values or PCA trend checks.

## Related tools

- **speaq 2.0** (Provides the differential analysis routine whose per-feature linear-model p-values are Bonferroni-corrected) — https://github.com/beirnaert/speaq
- **speaq R package** (CRAN distribution used in R to load spectra, build the feature matrix, and run scaling, imputation, and differential analysis) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the linear models and Bonferroni adjustment are computed) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm)); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The red-vs-white wine analysis (rosé removed, Pareto-scaled matrix) must reproduce exactly 33 Bonferroni-significant features.
- The onion intake in mice analysis must yield exactly 9 Bonferroni-significant peaks, matching Fig 8 of the paper.
- The results file must contain one row per tested feature (e.g., all 677 onion features) with a valid Bonferroni-adjusted p-value column.
- Invariant check: every adjusted p-value is >= its raw p-value and the adjusted vector is monotonically consistent with the raw ordering.
- As a sanity check on the onion study, PCA on the feature matrix should show no consistent separation by diet group despite the 9 significant peaks.

## Limitations

- The Bonferroni correction is conservative; in the onion study it calls 9 significant peaks whose PCA score plot shows no consistent group separation by diet group.
- Upstream peak grouping/peak filling can drop values (e.g., 'The speaq 2.0 algorithm deletes one of the large peaks in the grouping step'), reducing the number of features that reach the testing stage.
- Severely shifted spectra compromise alignment results, which affects the features entering differential analysis.

## Evidence

- [abstract] wavelet peak-picking feeds grouping and filling, avoiding binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] speaq offers peak-based processing including differential analysis: "Peak based approach (new since v2.0)
	- peak picking (wavelets)
	- grouping
	- peak filling
	- scaling & imputations
	- differential analysis (also more than 2 groups)"
- [readme] README feature matrix construction step feeding downstream statistics: [evidence span withheld — anchor status: not_found]
- [intro] significant peaks queried against HMDB: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
- [supplementary] SilhouetR quality check on grouping: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
