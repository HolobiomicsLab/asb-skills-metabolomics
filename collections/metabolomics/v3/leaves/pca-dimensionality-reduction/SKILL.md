---
name: pca-dimensionality-reduction
description: Use when a metabolomics or 'omics feature table (samples × metabolites,
  e.g. 90 NMR-quantified metabolites across 3–8 biological replicates per cell population)
  needs unbiased dimensionality reduction and unsupervised visualization of sample-group
  clustering.
when_to_use_negative:
- The analysis goal is conditional dependence structure between metabolites (partial
  correlations) rather than unsupervised variance decomposition — that requires the
  GLASSO step, not PCA alone.
- The data are raw NMR spectra rather than a normalized metabolite concentration table
  — spectra must first be pre-processed (TopSpin, Chenomx peak identification) before
  PCA.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0092
tools:
- name: pcaglasso
  role: Python package whose 'pca_bundle' class runs PCA and visualizes PCA-related
    graphs showing loadings, scores, and explained variance
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the sklearn PCA implementation used to fit 10 components (first three
    retained) on the z-score-normalized metabolite table
- name: matplotlib
  role: Used to plot sample scores in the first three principal components, colored
    by cell population, and export pca_score_plot.png
- name: pandas
  role: Loads and normalizes the metabolite concentration table (mean- and stddev-normalization)
    prior to PCA
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:fb25b831e43d2459fd26b9d9b79d5fd405c4a7f894903e406910cdb9c0d90184
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pca-dimensionality-reduction

## Summary

Unsupervised principal component analysis over mean- and stddev-normalized NMR metabolomics data (e.g., 90 metabolites measured by Chenomx from 1H-NMR spectra of hESC-derived neural stages), used to reduce dimensionality and demonstrate cell-type separation prior to GLASSO network overlay.

## When to use

Use when a metabolomics or 'omics feature table (samples × metabolites, e.g. 90 NMR-quantified metabolites across 3–8 biological replicates per cell population) needs unbiased dimensionality reduction and unsupervised visualization of sample-group clustering.

## When NOT to use

- The analysis goal is conditional dependence structure between metabolites (partial correlations) rather than unsupervised variance decomposition — that requires the GLASSO step, not PCA alone.
- The data are raw NMR spectra rather than a normalized metabolite concentration table — spectra must first be pre-processed (TopSpin, Chenomx peak identification) before PCA.

## Inputs

- Metabolite concentration table (n = 90 metabolites from Supplementary Table S1, quantified by Chenomx from 1H-NMR spectra acquired on a Bruker Avance 800 MHz spectrometer, 3–8 biological replicates per population)
- Sample metadata mapping each sample to its cell population (hESC, EB, rosettes, hNPC, neurons)

## Outputs

- pca_scores.csv — sample scores table for the retained principal components
- pca_score_plot.png — 3-component score plot colored by the five cell populations
- PCA loadings, sample scores, and explained variance per retained component (via the pca_bundle class)

## How to apply

Load the metabolite concentration table into a pandas DataFrame and normalize each metabolite feature by its mean and standard deviation (z-score scaling) as specified for the MetaboLINK PCA step. Fit PCA using scikit-learn, configured to produce 10 components of which the first three are retained, wrapped in the 'pca_bundle' class so that loadings, sample scores, and explained variance are captured per component. Plot the samples in the first three principal components with matplotlib, coloring points by cell population (hESC, EB, rosettes, hNPC, neurons) to confirm unsupervised clustering. The PCA loading scores are subsequently used to color GLASSO network nodes (MetaboLINK) and, in per-transition runs over sequential cell-type pairs (hESC-EB, EB-Rosettes, Rosettes-hNPC, hNPC-Neurons), to associate metabolites with one of the two cell types in a binary classification setup.

## Related tools

- **pcaglasso** (Python package whose 'pca_bundle' class runs PCA and visualizes PCA-related graphs showing loadings, scores, and explained variance) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the sklearn PCA implementation used to fit 10 components (first three retained) on the z-score-normalized metabolite table)
- **matplotlib** (Used to plot sample scores in the first three principal components, colored by cell population, and export pca_score_plot.png)
- **pandas** (Loads and normalizes the metabolite concentration table (mean- and stddev-normalization) prior to PCA)

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; X = (df - df.mean()) / df.std(); scores = PCA(n_components=10).fit_transform(X)
```

## Evaluation signals

- The PCA scores plot must clearly separate the five cell-type sample groups (hESC, EB, rosettes, hNPC, neurons) in an unsupervised manner, matching the paper's reported result that 'PCA clearly separates the cell types'.
- PCA is fit with 10 components and exactly the first three are retained for downstream use, per the MetaboLINK specification.
- Each metabolite feature must be normalized by its mean and standard deviation (z-score) before fitting; skipping normalization invalidates comparability with the paper's setup.
- pca_scores.csv and pca_score_plot.png exist and the scores table contains one row per sample with the three retained component scores.
- Explained variance is captured for each retained component via the pca_bundle class.

## Limitations

- The article does not state software versions or additional PCA parameter settings beyond producing 10 components, limiting exact reproducibility.
- No accession or availability statement is given for the metabolomics dataset analyzed (hESC, hNPC, neuron samples), so the exact input table may need reconstruction from Supplementary Table S1.
- PCA captures linear structure only; interpretation is tied to the small-n NMR metabolomics setting (3–8 replicates per population) and may not transfer to larger, noisier 'omics datasets without revalidation.

## Evidence

- [other] PCA of the metabolomics data separated the neural developmental stages, with each cell lineage clustering independently: [evidence span withheld — anchor status: not_found]
- [methods] We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three.: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] The class 'pca_bundle' runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance.: [evidence span withheld — anchor status: not_found]
- [readme] A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [methods] All plotting is done using matplotlib: "All plotting is done using matplotlib"
