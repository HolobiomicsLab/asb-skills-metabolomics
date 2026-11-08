---
name: kmeans-spatial-segmentation
description: Use when segmenting imzML mass spectrometry imaging pixels into spatially
  coherent regions via k-means, and when Silhouette scores must rank clusterings or
  compare hyperparameter search strategies.
when_to_use_negative:
- Input is raw or intensity-matrix MSI data that has not been dimensionality-reduced
  — k-means must run on the PCA+t-SNE embedded pixels per this pipeline, not directly
  on full spectral peaks
- The goal is peptide identification or LC-MS/MS confirmation of MSI annotations —
  that is MSIght's annotation/co-registration workflow, not k-means spatial segmentation
- Clusters are expected to have highly non-convex shapes where k-means assumptions
  fail and hierarchical or bottom-up clustering would be more appropriate (MSIght
  also exposes bottom_up_hierarchical_clustering-style scoring)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSIght
  role: Provides the clustering pipeline (MSIght.refactor_segment.cluster_msi) and
    scored k-means clustering with Silhouette reporting (kmeans_cluster_msi_scored_w_csv)
    over t-SNE-embedded MSI pixels
  repo: github.com/lingjunli-research/MSIght
- name: Python
  role: Language in which MSIght and the k-means/Silhouette segmentation workflow
    are implemented and distributed (pip install msight)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:25e310e94dbc61314e8201a94d0b3c7770e16c26917e324d3dd64d7f690e840b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kmeans-spatial-segmentation

## Summary

K-means spatial segmentation of MSI pixel data embedded by PCA+t-SNE, using Silhouette scores to score and select segmentation quality — implemented in MSIght's clustering pipeline (cluster_msi) and used to benchmark t-SNE hyperparameter optimization strategies (exhaustive, random, Bayesian).

## When to use

Use when segmenting imzML mass spectrometry imaging pixels into spatially coherent regions via k-means, and when Silhouette scores must rank clusterings or compare hyperparameter search strategies.

## When NOT to use

- Input is raw or intensity-matrix MSI data that has not been dimensionality-reduced — k-means must run on the PCA+t-SNE embedded pixels per this pipeline, not directly on full spectral peaks
- The goal is peptide identification or LC-MS/MS confirmation of MSI annotations — that is MSIght's annotation/co-registration workflow, not k-means spatial segmentation
- Clusters are expected to have highly non-convex shapes where k-means assumptions fail and hierarchical or bottom-up clustering would be more appropriate (MSIght also exposes bottom_up_hierarchical_clustering-style scoring)

## Inputs

- imzML mass spectrometry imaging dataset (intensity matrix over pixel coordinates)
- t-SNE-embedded 2-component pixel representation (from PCA-100 preprocessing)
- k-means cluster count (k=8 in the reference workflow)

## Outputs

- Per-pixel cluster/segmentation labels for the MSI image
- Silhouette score per clustering/parameter set
- Consolidated labels and Silhouette score table written to clustered_segmentation_results.csv

## How to apply

Load and preprocess the imzML MSI data with MSIght.refactor_common_functions.load_and_preprocess_imzml and create_intensity_matrix, then apply Gaussian smoothing (sigma=1.0, structuring element size 3). Reduce dimensionality via PCA to 100 components followed by t-SNE to 2 components (initial settings perplexity=30, learning_rate=200, n_iter=500) using MSIght.refactor_common_functions.apply_dimensionality_reduction. Cluster the t-SNE-embedded pixels with k-means into 8 clusters via MSIght.refactor_cluster_optimize.kmeans_cluster_msi_scored_w_csv, computing a Silhouette score per segmentation to quantify spatial segregation quality. To tune t-SNE hyperparameters (perplexity 5–50, learning rate 200–500, iterations 500–1500), run exhaustive grid, random (50 samples), and Bayesian (100 iterations) search over the same space and compare strategies by their best Silhouette score. Write cluster labels and the Silhouette score table to clustered_segmentation_results.csv.

## Related tools

- **MSIght** (Provides the clustering pipeline (MSIght.refactor_segment.cluster_msi) and scored k-means clustering with Silhouette reporting (kmeans_cluster_msi_scored_w_csv) over t-SNE-embedded MSI pixels) — github.com/lingjunli-research/MSIght
- **Python** (Language in which MSIght and the k-means/Silhouette segmentation workflow are implemented and distributed (pip install msight))

## Examples

```
from MSIght.refactor_segment import cluster_msi
cluster_msi(tsne_embedded_pixels, n_clusters=8)  # Run the clustering pipeline
```

## Evaluation signals

- Silhouette score is computed for every segmentation result (each k-means run / parameter combination) and the best strategy's score confirms the optimal t-SNE parameter combination (perplexity 5–50, learning rate 200–500, iterations 500–1500)
- The clustering strategy comparison (exhaustive vs. random vs. Bayesian) should yield similar Silhouette values, as reported in Figure 2A and Supplementary Figures S2–S4
- Consolidated output must be a clustered_segmentation_results.csv containing clustering labels plus the Silhouette score table
- Embedded pixels must be 2-component t-SNE coordinates derived from 100 PCA components of Gaussian-smoothed (sigma=1.0, structuring element size 3) intensity data
- Cluster count is fixed at k=8 for the reference MSI segmentation

## Limitations

- K-means segmentation quality depends on t-SNE hyperparameters; the reference workflow reports similar Silhouette values across exhaustive, random, and Bayesian search, so no single search strategy is decisively better and exhaustive search is costly over the full grid
- Requires prior preprocessing (Gaussian smoothing sigma=1.0, PCA to 100 components) — applying k-means to unsmoothed or full-dimensional MSI data is outside the validated workflow
- MSIght is distributed as a Python package requiring pip installation and a Jupyter notebook walkthrough; the article notes existing tools often require expertise in both MSI and image analysis, and MSIght's workflow (pre-processing, registration, LC-MS/MS confirmation) must otherwise be assembled manually

## Evidence
