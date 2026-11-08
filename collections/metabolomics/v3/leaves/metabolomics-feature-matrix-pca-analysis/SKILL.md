---
name: metabolomics-feature-matrix-pca-analysis
description: Use when you already have a grouped metabolomics feature matrix (e.g.
  from speaq 2.0 peak detection on 1D NMR spectra) and want an unsupervised check
  that samples separate by a known class factor such as wine color.
when_to_use_negative:
- Input is raw spectra or an ungrouped peak list — the matrix must already come from
  peak picking, grouping and BuildFeatureMatrix
- You need supervised class prediction rather than unsupervised exploratory separation
  — PCA does not provide class assignment
- You need metabolite concentration quantification — use speaq's quantitation (BW-ratio)
  workflow instead
edam_operation: http://edamontology.org/operation_2939
tools:
- name: speaq R package
  role: Produces the feature matrix (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix)
    that is scaled and subjected to PCA
  repo: https://cran.r-project.org/package=speaq
- name: speaq 2.0
  role: Peak-picking (wavelets) and grouping workflow whose output feeds the PCA analysis
  repo: https://github.com/beirnaert/speaq
- name: R
  role: Execution environment; prcomp is used for PCA and base graphics for the score
    plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:8d5c9f827efb250b9837d31a5f893e846694fe54b645e059da7cfd58115c029b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolomics-feature-matrix-pca-analysis

## Summary

Runs PCA (prcomp) on a Pareto-scaled, mean-centered feature/sample/peak-value matrix produced by the speaq 2.0 peak-picking and grouping workflow, and plots PC1 vs PC2 scores colored by sample class to reveal group separation.

## When to use

Use when you already have a grouped metabolomics feature matrix (e.g. from speaq 2.0 peak detection on 1D NMR spectra) and want an unsupervised check that samples separate by a known class factor such as wine color.

## When NOT to use

- Input is raw spectra or an ungrouped peak list — the matrix must already come from peak picking, grouping and BuildFeatureMatrix
- You need supervised class prediction rather than unsupervised exploratory separation — PCA does not provide class assignment
- You need metabolite concentration quantification — use speaq's quantitation (BW-ratio) workflow instead

## Inputs

- Grouped feature/sample/peak-value matrix produced by speaq 2.0 (getWaveletPeaks -> PeakGrouper -> BuildFeatureMatrix), e.g. from Winedata 1D NMR spectra
- Sample class factor (e.g. wine.color: red, white, rosé) aligned with the matrix rows/columns

## Outputs

- PCA model object from R's prcomp (scores, loadings, explained variance)
- PC1 vs PC2 score plot colored by sample class (wine_pca_score_plot.png)

## How to apply

Start from the feature matrix built by speaq 2.0's BuildFeatureMatrix from wavelet-detected, grouped peaks (getWaveletPeaks + PeakGrouper), with an accompanying class factor (e.g. Winedata$wine.color with red, white, rosé groups). Apply Pareto scaling — divide each variable (column) by the square root of its standard deviation — combined with mean centering, so both abundant and low-abundance metabolites contribute comparably to the components. Run R's prcomp on the scaled matrix, then plot the PC1 versus PC2 score vectors with points colored by the class factor. For the wine dataset this reproduction should show Principal Component 2 clearly separating red wines from white and rosé wines, matching the S1 Fig result; save the score plot (e.g. wine_pca_score_plot.png) as the deliverable.

## Related tools

- **speaq R package** (Produces the feature matrix (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) that is scaled and subjected to PCA) — https://cran.r-project.org/package=speaq
- **speaq 2.0** (Peak-picking (wavelets) and grouping workflow whose output feeds the PCA analysis) — https://github.com/beirnaert/speaq
- **R** (Execution environment; prcomp is used for PCA and base graphics for the score plot)

## Examples

```
pca <- prcomp(sweep(Features, 2, apply(Features,2,sd), function(x,y) x/sqrt(y)), center=TRUE); plot(pca$x[,1], pca$x[,2], col=class.factor)
```

## Evaluation signals

- The Pareto scaling reduces each column by the square root of its standard deviation and centers it — verify column variances are proportional to original standard deviations
- PC2 scores should clearly separate red wine samples from white and rosé wine samples on the wine dataset, reproducing S1 Fig
- Number of principal components should equal min(n_samples, n_features) with monotonically decreasing explained variance
- Score plot colors must map 1:1 onto the wine.color factor levels (red, white, rosé) with no misaligned samples
- Figure file wine_pca_score_plot.png is produced with PC1 on one axis and PC2 on the other

## Limitations

- PCA is unsupervised: separation along PC2 indicates but does not statistically test group differences
- The downstream grouping/peak-filling step can delete or miss peaks (e.g. the algorithm deletes one of the large peaks in the grouping step), which can distort the feature matrix feeding PCA
- Binning-based alternatives lose information compared to the peak-based feature matrix; results are sensitive to the scaling choice (here Pareto scaling with mean centering)

## Evidence

- [supplementary] supplementary dataset finding: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [readme] README minimal example: [evidence span withheld — anchor status: not_found]
- [abstract] abstract workflow description: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README pipeline functions: "peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)"
