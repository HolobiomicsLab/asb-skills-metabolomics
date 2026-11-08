---
name: pca-score-plot-visualization
description: 'Use when a grouped NMR peak feature matrix (samples × peaks) must be
  checked for class structure: Pareto-scale and mean-center it, run PCA, and inspect
  the PC1–PC2 score plot colored by wine color for red vs white/rosé separation.'
when_to_use_negative:
- Input is raw, ungrouped spectra — this skill starts from the grouped feature matrix;
  run speaq 2.0 peak picking and grouping (getWaveletPeaks → PeakGrouper) first.
- The goal is quantitation (BW-ratio) or spectral alignment (CluPA) rather than exploratory
  visualization of group separation.
- No sample class/color annotation exists, so the score plot cannot be colored and
  group separation cannot be judged.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: speaq 2.0
  role: Produces the feature/sample/peak-value matrix (getWaveletPeaks, PeakGrouper,
    BuildFeatureMatrix) that the PCA score plot is computed from; distributed via
    the speaq R package.
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R package providing the wine demo data (Winedata) and the peak-based processing
    functions feeding BuildFeatureMatrix; the feature matrix it returns 'can be processed
    further (scaling, transformations) or analysed with ... PCA'.
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime for the analysis; prcomp() computes the PCA whose PC1/PC2 scores are
    plotted and colored by wine color.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:a10364063f2c6b7a5a8930b8891896e1131b13137829db589398b3eafab6a5dd
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pca-score-plot-visualization

## Summary

Runs PCA on a Pareto-scaled, mean-centered 1D NMR feature/sample/peak-value matrix produced by the speaq 2.0 workflow and plots the PC1 vs PC2 score plot colored by wine color, demonstrating that red wines separate from white and rosé wines along Principal Component 2 (S1 Fig).

## When to use

Use when a grouped NMR peak feature matrix (samples × peaks) must be checked for class structure: Pareto-scale and mean-center it, run PCA, and inspect the PC1–PC2 score plot colored by wine color for red vs white/rosé separation.

## When NOT to use

- Input is raw, ungrouped spectra — this skill starts from the grouped feature matrix; run speaq 2.0 peak picking and grouping (getWaveletPeaks → PeakGrouper) first.
- The goal is quantitation (BW-ratio) or spectral alignment (CluPA) rather than exploratory visualization of group separation.
- No sample class/color annotation exists, so the score plot cannot be colored and group separation cannot be judged.

## Inputs

- speaq 2.0 feature/sample/peak-value matrix (BuildFeatureMatrix output: samples × grouped peaks)
- 1D NMR spectra matrix, ppm vector, and sample class factor (e.g. Winedata$spectra, Winedata$ppm, Winedata$wine.color) if the feature matrix must first be built from raw spectra

## Outputs

- PCA score plot of PC1 vs PC2 with samples colored by wine type (saved as wine_pca_score_plot.png)
- prcomp PCA object (scores and loadings) computed on the Pareto-scaled, mean-centered feature matrix

## How to apply

Load the grouped feature/sample/peak-value matrix produced upstream by speaq 2.0 — wavelet peak picking (getWaveletPeaks) followed by PeakGrouper grouping and BuildFeatureMatrix — which deliberately avoids the binning step so per-peak information is retained. Apply Pareto scaling to the feature matrix: each variable (grouped peak) is divided by the square root of its standard deviation, with mean centering. Run principal component analysis on this Pareto-scaled, centered matrix with R's prcomp. Plot PC1 (x-axis) versus PC2 (y-axis) scores, coloring each sample by its wine color class (red, white, rosé). Judge success by whether PC2 clearly separates red wines from the white and rosé wines, matching the paper's S1 Fig result, and save the figure as wine_pca_score_plot.png.

## Related tools

- **speaq 2.0** (Produces the feature/sample/peak-value matrix (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) that the PCA score plot is computed from; distributed via the speaq R package.) — https://github.com/beirnaert/speaq
- **speaq R package** (R package providing the wine demo data (Winedata) and the peak-based processing functions feeding BuildFeatureMatrix; the feature matrix it returns 'can be processed further (scaling, transformations) or analysed with ... PCA'.) — https://cran.r-project.org/package=speaq
- **R** (Runtime for the analysis; prcomp() computes the PCA whose PC1/PC2 scores are plotted and colored by wine color.)

## Examples

```
pc <- prcomp(scale(Features, center=TRUE, scale=sqrt(apply(Features,2,sd)))); plot(pc$x[,1], pc$x[,2], col=Winedata$wine.color)
```

## Evaluation signals

- PC2 (y-axis) clearly separates red wines from white and rosé wines, reproducing the S1 Fig result.
- Scaling is Pareto — each variable divided by the square root of its standard deviation — plus mean centering, not full auto-scaling.
- Scores plotted are exactly PC1 vs PC2 from R's prcomp, not later components.
- Points are colored by the wine color classes (red, white, rosé) and the figure is saved as wine_pca_score_plot.png.
- The PCA input is the grouped peak feature matrix, not a binned/bucketed spectral matrix.

## Limitations

- Grouping artifacts propagate downstream: the paper notes the algorithm deletes one of the large peaks in the grouping step and that peak filling may not always mitigate it; the optional SilhouetR function calculates silhouette values per group to detect such problems.
- The reported red vs white/rosé separation along PC2 is a result for the wine dataset (S1 Fig); separation is not guaranteed for other datasets or class variables.
- Only the first two PCs are visualized; separation confined to higher components will be missed.

## Evidence

- [supplementary] The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines.: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [readme] this featurematrix can be processed further (scaling, transformations) or analysed with the statistical tools of interest like PCA.: [evidence span withheld — anchor status: not_found]
- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group.: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
