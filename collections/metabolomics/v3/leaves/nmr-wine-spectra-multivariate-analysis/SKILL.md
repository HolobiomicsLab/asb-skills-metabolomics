---
name: nmr-wine-spectra-multivariate-analysis
description: Use when a peak-based NMR feature matrix (e.g. from speaq 2.0 peak picking
  and grouping) is available and the goal is to test whether samples separate by a
  known class label such as wine colour via PCA.
when_to_use_negative:
- Input is raw 1D NMR spectra that have not yet gone through peak picking, grouping
  and feature matrix construction — run the speaq 2.0 workflow first
- The analysis target is quantitation or spectral alignment rather than between-sample
  class separation
- Data are already scaled (e.g. autoscaled) — re-applying Pareto scaling would distort
  the matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Produces the peak-based feature matrix (peak picking, grouping, feature matrix
    building) that is scaled and subjected to PCA
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Provides getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, scaling/imputation
    and visualization functions used to prepare the matrix for PCA
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment; prcomp is used to run PCA on the Pareto-scaled matrix
    and base plotting to draw the score plot
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:70a814ef529758f78aa252a3fe4755bb84a9299a0f005165998e2e1b49f8c3ba
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
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# nmr-wine-spectra-multivariate-analysis

## Summary

Pareto scaling (dividing each variable by the square root of its standard deviation) with mean centering of a speaq 2.0-derived feature/sample/peak-value matrix, followed by PCA (prcomp) and score plotting to reveal sample groupings such as red vs white/rosé wines.

## When to use

Use when a peak-based NMR feature matrix (e.g. from speaq 2.0 peak picking and grouping) is available and the goal is to test whether samples separate by a known class label such as wine colour via PCA.

## When NOT to use

- Input is raw 1D NMR spectra that have not yet gone through peak picking, grouping and feature matrix construction — run the speaq 2.0 workflow first
- The analysis target is quantitation or spectral alignment rather than between-sample class separation
- Data are already scaled (e.g. autoscaled) — re-applying Pareto scaling would distort the matrix

## Inputs

- Grouped feature/sample/peak-value matrix from the speaq 2.0 peak-picking and grouping workflow (BuildFeatureMatrix output)
- Class factor/vector of sample labels (e.g. wine.color: red, white, rosé)
- ppm vector and spectra matrix (Winedata$spectra, Winedata$ppm) if starting from raw spectra

## Outputs

- PCA model (prcomp object) of the Pareto-scaled feature matrix
- PC1 vs PC2 score plot coloured by wine type (wine_pca_score_plot.png)

## How to apply

Load the grouped wine feature/sample/peak-value matrix produced by the speaq 2.0 workflow (getWaveletPeaks -> PeakGrouper -> BuildFeatureMatrix). Apply Pareto scaling — each variable divided by the square root of its standard deviation — together with mean centering to the feature matrix, which balances the influence of large and small peaks without full autoscaling's noise amplification. Run principal component analysis on the scaled matrix using R's prcomp. Plot PC1 versus PC2 scores, colouring samples by wine type (red, white, rosé), and save the figure (e.g. wine_pca_score_plot.png).

## Related tools

- **speaq 2.0** (Produces the peak-based feature matrix (peak picking, grouping, feature matrix building) that is scaled and subjected to PCA) — https://github.com/beirnaert/speaq
- **speaq R package** (Provides getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, scaling/imputation and visualization functions used to prepare the matrix for PCA) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment; prcomp is used to run PCA on the Pareto-scaled matrix and base plotting to draw the score plot) — https://cran.r-project.org/package=speaq

## Examples

```
peaks <- getWaveletPeaks(Y.spec=spectra.matrix, X.ppm=ppm.vector); groups <- PeakGrouper(Y.peaks=peaks); Features <- BuildFeatureMatrix(Y.data=groups); pca <- prcomp(scale(Features$data, scale=sqrt(apply(Features$data,2,sd))))
```

## Evaluation signals

- PC2 of the PCA score plot clearly separates red wines from white and rosé wines, reproducing the reported S1 Fig result
- The feature matrix is Pareto scaled and centered (each column divided by sqrt(sd) then mean-centered) before prcomp is called
- The saved score plot (wine_pca_score_plot.png) has points coloured by wine type (red, white, rosé) matching the class factor
- The input matrix traces back to the speaq 2.0 pipeline (getWaveletPeaks -> PeakGrouper -> BuildFeatureMatrix) rather than a binned matrix

## Limitations

- The wine PCA result is reported as a supplementary figure (S1 Fig); the article provides no numeric separation metric, so verification is visual
- Pareto scaling is chosen over binning-based preprocessing; the article argues binning suffers intrinsic information loss and difficult bin boundary definition in crowded spectra
- Severely shifted spectra can degrade downstream steps — the article notes spectral alignment results are 'not always optimal when dealing with severely shifted spectra'

## Evidence

- [supplementary] PCA of the Pareto-scaled and centered wine feature matrix shows a clear difference between red wines on one side and white and rosé wines on the other (S1 Fig).: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [readme] README minimal example shows the exact function chain from Winedata to the feature matrix that feeds PCA.: [evidence span withheld — anchor status: not_found]
- [abstract] speaq 2.0 avoids the binning step by peak-picking followed by grouping, yielding the feature/sample/peak-value matrix used here.: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
