---
name: pca-loadings-and-explained-variance-interpretation
description: Use when a small, feature-rich metabolomics or omics table (samples x
  metabolites) needs unbiased dimensionality reduction and you must inspect per-component
  loadings, sample scores, and explained variance to judge which components capture
  significant variance.
when_to_use_negative:
- When the data are raw, unnormalized spectra or peak lists — z-score normalization
  by mean and stddev must precede PCA, so an unprocessed matrix will bias components
  toward high-abundance metabolites
- When the goal is supervised classification of cell types — PCA is unsupervised and
  only demonstrates natural clustering, not predictive separation
- When the GLASSO sub-network identification step (MetaboLINK) is the target analysis
  — this skill supplies the PCA scores that get overlaid, but does not itself run
  GraphLassoCV
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Provides the PCA (and sklearn.covariance GLASSO) implementations used to fit
    the 10-component model on the normalized dataset
- name: pcaglasso
  role: Python package whose 'pca_bundle' class runs PCA and exposes loadings, scores,
    and explained variance; GLASSO graphs are overlaid with PCA scores to identify
    significant sub-networks
  repo: github.com/jlichtarge/pcaGLASSO
- name: matplotlib
  role: Used for all plotting, including score plots of samples in the first three
    principal components
- name: pandas
  role: Loads and holds the metabolite concentration table and writes the scores table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:2f259812362d8349982d10ce6e6aa3566cdc7006dec11767dbc3c1dd2471b95e
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# PCA loadings and explained variance interpretation

## Summary

Runs PCA on mean- and stddev-normalized metabolite abundance data (e.g., 90 metabolites from 1H-NMR/Chenomx measurements of hESC-derived neural populations) via scikit-learn, producing 10 components with the first three retained, and uses the 'pca_bundle' class to expose loadings, sample scores, and explained variance for visualization and interpretation.

## When to use

Use when a small, feature-rich metabolomics or omics table (samples x metabolites) needs unbiased dimensionality reduction and you must inspect per-component loadings, sample scores, and explained variance to judge which components capture significant variance.

## When NOT to use

- When the data are raw, unnormalized spectra or peak lists — z-score normalization by mean and stddev must precede PCA, so an unprocessed matrix will bias components toward high-abundance metabolites
- When the goal is supervised classification of cell types — PCA is unsupervised and only demonstrates natural clustering, not predictive separation
- When the GLASSO sub-network identification step (MetaboLINK) is the target analysis — this skill supplies the PCA scores that get overlaid, but does not itself run GraphLassoCV

## Inputs

- pandas DataFrame of metabolite concentrations (e.g., n = 90 metabolites from Supplementary Table S1, quantified by Chenomx from 1H-NMR spectra, 3-8 biological replicates per population)
- Sample-to-population label mapping (e.g., hESC, EB, rosettes, hNPCs, neurons) for coloring score plots

## Outputs

- PCA sample scores for the first three components (exportable as pca_scores.csv)
- Per-component PCA loadings identifying metabolites driving each component
- Explained variance values for each retained component
- Score plot of samples in the first three principal components (pca_score_plot.png)

## How to apply

Normalize each metabolite feature by its mean and standard deviation (z-score scaling), as specified for the MetaboLINK PCA step. Fit PCA with the scikit-learn implementation configured to produce 10 components, of which the first three are used downstream. Capture the PCA loadings, sample scores, and explained variance for each retained component using the 'pca_bundle' class from the pcaglasso package. Plot samples in the first three principal components with matplotlib (e.g., coloring hESC, EB, rosette, hNPC, and neuron samples) to visualize unsupervised separation; loadings identify which metabolites drive each component and can be overlaid with GLASSO graphs to locate significant sub-networks. Justification: the paper ran PCA over the mean- and stddev-normalized dataset to produce 10 components and retained the first three, and states that pca_bundle 'allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance.'

## Related tools

- **scikit-learn** (Provides the PCA (and sklearn.covariance GLASSO) implementations used to fit the 10-component model on the normalized dataset)
- **pcaglasso** (Python package whose 'pca_bundle' class runs PCA and exposes loadings, scores, and explained variance; GLASSO graphs are overlaid with PCA scores to identify significant sub-networks) — github.com/jlichtarge/pcaGLASSO
- **matplotlib** (Used for all plotting, including score plots of samples in the first three principal components)
- **pandas** (Loads and holds the metabolite concentration table and writes the scores table)

## Examples

```
from pcaglasso import pca_bundle; pb = pca_bundle(X, n_components=10); pb.plot_scores(labels=populations); pb.scores.to_csv('pca_scores.csv')
```

## Evaluation signals

- The input matrix was z-score normalized (mean- and stddev-normalized) before fitting; verify feature means ~0 and stddevs ~1
- PCA was configured to produce 10 components, and only the first three are retained for downstream use — check n_components=10 and that loadings/scores/vectors for exactly 3 components are exported
- pca_scores.csv contains a scores column for each of the three retained components and one row per sample (3-8 biological replicates per population, five populations)
- The score plot colors points by the five cell populations (hESC, EB, rosettes, hNPCs, neurons) and shows each lineage clustering independently
- Explained variance is reported for each retained component, matching what pca_bundle captures

## Limitations

- PCA is unsupervised: separation of the five neural stages in the score plot confirms distinctness but assigns no statistical significance or class labels
- No software versions, explicit MetaboLINK parameter settings, or dataset accession/availability statement are given, which hinders exact reproducibility
- Only the first three of ten components are used; metabolites loading on components 4-10 are not interpreted in this step

## Evidence

- [methods] PCA over mean- and stddev-normalized dataset producing 10 components (first three used): "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] pca_bundle exposes loadings, scores, and explained variance: "The class ‘pca_bundle’ runs PCA and allows the easy visualization of PCA-related graphs, showing loadings, scores, and explained variance."
- [readme] GLASSO graphs overlaid with PCA scores: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
- [methods] scikit-learn implementations: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [intro] MetaboLINK combines PCA loadings with GLASSO: "Our method seamlessly combines principal component analysis (PCA) loadings with the graphical lasso (GLASSO) to reveal subnetworks in metabolomics datasets"
