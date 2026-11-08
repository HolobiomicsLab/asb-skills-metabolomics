---
name: pareto-scaling-data-normalization
description: Use when you have a peak-based feature/sample/peak-value matrix from
  NMR spectra (e.g. speaq 2.0 output) and need to equalize variable influence before
  PCA or multivariate statistics.
when_to_use_negative:
- The input is raw spectra rather than a peak-picked, grouped feature matrix — run
  getWaveletPeaks, PeakGrouper and BuildFeatureMatrix first
- The downstream method already performs internal scaling (e.g. prcomp with scale.=TRUE
  gives unit-variance scaling, not Pareto scaling)
- The goal is quantitation or spectral alignment rather than statistical analysis
  of the feature matrix
edam_operation: http://edamontology.org/operation_2939
tools:
- name: speaq R package
  role: Produces the feature matrix (peak picking, grouping, peak filling) that is
    Pareto scaled before statistical analysis
  repo: https://cran.r-project.org/package=speaq
- name: speaq 2.0
  role: Workflow framework whose feature/sample/peak-value matrix is the input to
    scaling and PCA
  repo: https://github.com/beirnaert/speaq
- name: R
  role: Environment in which scaling and prcomp-based PCA are executed
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:ad445203a347a5e0cab41a8d1e68dba84509d6a3a61d685d91de3090b775851e
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
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pareto-scaling-data-normalization

## Summary

Pareto scaling (dividing each variable by the square root of its standard deviation) with mean centering of a peak-based feature matrix, preparing NMR metabolomics data for PCA and other downstream statistical analysis.

## When to use

Use when you have a peak-based feature/sample/peak-value matrix from NMR spectra (e.g. speaq 2.0 output) and need to equalize variable influence before PCA or multivariate statistics.

## When NOT to use

- The input is raw spectra rather than a peak-picked, grouped feature matrix — run getWaveletPeaks, PeakGrouper and BuildFeatureMatrix first
- The downstream method already performs internal scaling (e.g. prcomp with scale.=TRUE gives unit-variance scaling, not Pareto scaling)
- The goal is quantitation or spectral alignment rather than statistical analysis of the feature matrix

## Inputs

- Grouped wine feature/sample/peak-value matrix from the speaq 2.0 peak-picking and grouping workflow (BuildFeatureMatrix output)
- Sample class factor (e.g. wine.color: red, white, rosé) for coloring/interpreting downstream score plots

## Outputs

- Pareto-scaled and mean-centered feature matrix
- PCA score plot (PC1 vs PC2, colored by wine type), e.g. wine_pca_score_plot.png

## How to apply

Start from the grouped feature matrix produced by the speaq 2.0 peak-picking (getWaveletPeaks) and grouping (PeakGrouper) steps, assembled via BuildFeatureMatrix. Apply mean centering to each variable (column), then divide each variable by the square root of its standard deviation — Pareto scaling, which sits between no scaling and unit-variance scaling and preserves large spectral peaks while letting smaller metabolite peaks contribute. Feed the scaled matrix directly into prcomp for PCA or other statistical tools in R. In the speaq 2.0 wine dataset, this scaling plus PC1-vs-PC2 score plotting (colored by wine color factor: red, white, rosé) reproduces the reported separation of red wines from white/rosé wines along PC2 (S1 Fig).

## Related tools

- **speaq R package** (Produces the feature matrix (peak picking, grouping, peak filling) that is Pareto scaled before statistical analysis) — https://cran.r-project.org/package=speaq
- **speaq 2.0** (Workflow framework whose feature/sample/peak-value matrix is the input to scaling and PCA) — https://github.com/beirnaert/speaq
- **R** (Environment in which scaling and prcomp-based PCA are executed) — https://cran.r-project.org/package=speaq

## Examples

```
X.scaled <- sweep(sweep(Features, 2, colMeans(Features), "-"), 2, sqrt(apply(Features, 2, sd)), "/"); pca <- prcomp(X.scaled)
```

## Evaluation signals

- Each scaled column has mean ~0 and standard deviation equal to the square root of its original standard deviation (Pareto, not unit variance)
- PC1-vs-PC2 score plot of the wine dataset separates red wines from white and rosé wines along PC2, matching the reported S1 Fig
- The scaling step is applied to the BuildFeatureMatrix output without dropping samples; the number of rows is preserved
- Score plot samples are colored by the wine color factor (red, white, rosé) so the class structure is directly inspectable

## Limitations

- The scaling is only as good as the upstream peak picking/grouping: speaq 2.0's grouping step can delete one of two large peaks, and peak filling does not always mitigate this (detectable via the optional SilhouetR silhouette diagnostics)
- Spectral alignment results may be suboptimal for severely shifted spectra, which propagates into the feature matrix being scaled

## Evidence

- [readme] scaling/transformations before PCA is standard: [evidence span withheld — anchor status: not_found]
- [supplementary] PCA separates wine colors along PC2: "The PCA score plot shows that Principal Component 2 clearly indicates a difference between red, white and rosé wines."
- [abstract] speaq delivered via R package: "speaq 2.0 is distributed through the existing speaq R package"
