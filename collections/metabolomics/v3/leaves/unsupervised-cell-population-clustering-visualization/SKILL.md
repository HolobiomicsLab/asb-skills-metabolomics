---
name: unsupervised-cell-population-clustering-visualization
description: Use when you have a metabolite (or other feature) concentration table
  with biological replicates per population and need to verify, without supervision,
  that the populations are biochemically distinct before downstream pathway/network
  analyses.
when_to_use_negative:
- The goal is supervised classification or biomarker selection rather than an unsupervised
  distinctness check — PCA does not use labels to optimize separation.
- The input is not a normalized feature-by-sample concentration table (e.g., raw spectra
  or FID files); spectra must first be processed via TopSpin and Chenomx peak identification.
- Only one or two sample groups with very few replicates are present, where unsupervised
  clustering cannot demonstrate distinctness.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: 'Defines the PCA step: PCA over mean- and stddev-normalized data producing
    10 components, first three used'
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Python package providing the 'pca_bundle' class that runs PCA and visualizes
    loadings, scores, and explained variance
  repo: github.com/jlichtarge/pcaGLASSO
- name: scikit-learn
  role: Provides the PCA implementation used to fit components on the z-score-scaled
    metabolite table
- name: matplotlib
  role: All plotting, including the three-component PCA score plot colored by cell
    population
- name: pandas
  role: Loads and holds the metabolite concentration table prior to normalization
    and PCA
- name: Chenomx
  role: Upstream peak identification producing the metabolite abundances that feed
    the PCA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:4554db293c63d28c4f6abb3395b79b597870c6e6a484c45e4e0ea35ba982f068
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chenomx
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# unsupervised-cell-population-clustering-visualization

## Summary

Unsupervised PCA of metabolite abundance tables (z-score scaled) to test whether biological sample groups—here the five hESC-derived neural stages (hESCs, EBs, rosettes, hNPCs, neurons)—separate into distinct clusters without labels, plus score-plot visualization.

## When to use

Use when you have a metabolite (or other feature) concentration table with biological replicates per population and need to verify, without supervision, that the populations are biochemically distinct before downstream pathway/network analyses.

## When NOT to use

- The goal is supervised classification or biomarker selection rather than an unsupervised distinctness check — PCA does not use labels to optimize separation.
- The input is not a normalized feature-by-sample concentration table (e.g., raw spectra or FID files); spectra must first be processed via TopSpin and Chenomx peak identification.
- Only one or two sample groups with very few replicates are present, where unsupervised clustering cannot demonstrate distinctness.

## Inputs

- Metabolite concentration table (e.g., n=90 metabolites from Supplementary Table S1, Chenomx-quantified from 1H-NMR, 3–8 biological replicates per population) as a pandas DataFrame
- Per-sample cell population labels (hESC, EB, rosettes, hNPC, neurons) for coloring points

## Outputs

- pca_scores.csv — sample scores for the retained principal components
- pca_score_plot.png — 3-component score plot colored by the five cell populations
- PCA loadings, sample scores, and explained variance for the retained components (captured by the 'pca_bundle' class)

## How to apply

Load the metabolite concentration table (e.g., 90 metabolites quantified by Chenomx from 1H-NMR spectra on a Bruker Avance 800 MHz spectrometer, 3–8 biological replicates per population) into a pandas DataFrame. Normalize each feature by mean and standard deviation (z-score scaling) as specified for the MetaboLINK PCA step. Fit PCA with the scikit-learn implementation configured to produce 10 components and retain the first three, wrapping the run in the 'pca_bundle' class to capture loadings, sample scores, and explained variance per component. Plot the samples in the first three principal components with matplotlib, coloring points by the five cell populations. The run succeeds only if the score plot separates the groups unsupervised—matching the paper's result that 'PCA clearly separates the cell types'.

## Related tools

- **MetaboLINK (PCA-GLASSO algorithm)** (Defines the PCA step: PCA over mean- and stddev-normalized data producing 10 components, first three used) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Python package providing the 'pca_bundle' class that runs PCA and visualizes loadings, scores, and explained variance) — github.com/jlichtarge/pcaGLASSO
- **scikit-learn** (Provides the PCA implementation used to fit components on the z-score-scaled metabolite table)
- **matplotlib** (All plotting, including the three-component PCA score plot colored by cell population)
- **pandas** (Loads and holds the metabolite concentration table prior to normalization and PCA)
- **Chenomx** (Upstream peak identification producing the metabolite abundances that feed the PCA)

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(n_components=10); scores = pca.fit_transform(df_zscored)[:, :3]
```

## Evaluation signals

- The PCA score plot must clearly separate the five cell-type sample groups (hESC, EB, rosettes, hNPC, neurons) in an unsupervised manner, matching the paper's reported result that 'PCA clearly separates the cell types'.
- PCA was fit over the mean- and stddev-normalized (z-score) dataset and configured to produce 10 components, with the first three retained — verify component count and scaling in the fitted object.
- The 'pca_bundle' output contains loadings, sample scores, and explained variance for each retained component — check all three attributes are populated.
- pca_scores.csv contains one row per biological replicate (3–8 per population) with score columns for the retained components.
- Explained variance values for the retained components are reported and are consistent with the loadings/scores reconstruction.

## Limitations

- No software versions or specific PCA parameter settings beyond 10 components / first three are stated in the article, hindering exact reproducibility.
- No accession or availability statement is given for the metabolomics dataset analyzed, so the exact input table may need to be reconstructed from Supplementary Table S1.
- PCA separation is qualitative ('clusters independently'); the article does not report quantitative cluster-separation metrics for this validation step.

## Evidence

- [methods] PCA over mean- and stddev-normalized dataset producing 10 components (first three used): "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] pca_bundle class captures loadings, scores, explained variance: "The class ‘pca_bundle’ runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance."
- [intro] MetaboLINK validated on hESC-derived neural cell populations: "We validated the effectiveness of PCA-GLASSO (referred to as MetaboLINK from now on) for metabolomics analysis of human embryonic stem cell (hESC)-derived neural cell populations."
- [readme] pcaglasso package purpose: "Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
