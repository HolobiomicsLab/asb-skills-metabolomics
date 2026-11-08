---
name: mass-spectrometry-imaging-clustering
description: Use when imzML MSI pixel spectra need unsupervised segmentation into
  tissue clusters, especially when t-SNE embedding parameters (perplexity, learning
  rate, iterations) must be tuned and compared via Silhouette score.
when_to_use_negative:
- The goal is peptide identification confidence from untargeted LC-MS/MS confirmation
  rather than unsupervised tissue segmentation of MSI pixels
- Input is not spatially resolved imaging data (e.g., bulk LC-MS/MS or already-extracted
  feature tables), since the skill depends on per-pixel imzML spectra and ion-image
  context
- A supervised or histology-guided segmentation is required — this skill is unsupervised
  k-means on t-SNE embeddings and does not by itself use H&E co-registration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3344
tools:
- name: MSIght
  role: 'Provides the full clustering pipeline: imzML loading/preprocessing, PCA+t-SNE
    dimensionality reduction, k-means clustering with Silhouette scoring, and hyperparameter
    optimization modules (refactor_segment, refactor_cluster_optimize)'
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language chosen for popularity and accessibility so non-expert
    users can run the clustering workflow
- name: Jupyter
  role: Distributable notebook that explains the analysis pipeline (including clustering)
    step-by-step for programming novices
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:719cbf7a8e47698248e20f9c29b2af0b76f4b027b2424d535ed21c9aae65c9eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-imaging-clustering

## Summary

Unsupervised segmentation of MALDI-MSI pixel spectra by embedding preprocessed intensity data via PCA-to-t-SNE and k-means clustering, with Silhouette-score-based comparison of t-SNE hyperparameter optimization strategies (exhaustive, random, Bayesian) as implemented in MSIght.

## When to use

Use when imzML MSI pixel spectra need unsupervised segmentation into tissue clusters, especially when t-SNE embedding parameters (perplexity, learning rate, iterations) must be tuned and compared via Silhouette score.

## When NOT to use

- The goal is peptide identification confidence from untargeted LC-MS/MS confirmation rather than unsupervised tissue segmentation of MSI pixels
- Input is not spatially resolved imaging data (e.g., bulk LC-MS/MS or already-extracted feature tables), since the skill depends on per-pixel imzML spectra and ion-image context
- A supervised or histology-guided segmentation is required — this skill is unsupervised k-means on t-SNE embeddings and does not by itself use H&E co-registration

## Inputs

- imzML mass spectrometry imaging file (pixel spectra with spatial coordinates)
- Pixel-by-peak intensity matrix created by MSIght.refactor_common_functions.create_intensity_matrix
- t-SNE hyperparameter search space: perplexity 5–50, learning rate 200–500, iterations 500–1500
- Cluster count (k-means k=8 in the published workflow)

## Outputs

- Cluster label per MSI pixel (8-cluster tissue segmentation)
- Silhouette score per clustering run and per optimization strategy (exhaustive, random, Bayesian)
- Consolidated label and score table written to clustered_segmentation_results.csv

## How to apply

Load and preprocess the imzML file with MSIght.refactor_common_functions.load_and_preprocess_imzml and create_intensity_matrix to build the pixel-by-peak intensity matrix. Apply Gaussian smoothing (sigma=1.0, structuring element size 3), then reduce dimensionality with PCA to 100 components followed by t-SNE to 2 components (initial settings perplexity=30, learning_rate=200, n_iter=500) via MSIght.refactor_common_functions.apply_dimensionality_reduction. Cluster the 2D embedding into 8 clusters with k-means (MSIght.refactor_cluster_optimize.kmeans_cluster_msi_scored_w_csv) and compute a Silhouette score per segmentation. To tune t-SNE, search the space of perplexity 5–50, learning rate 200–500, and iterations 500–1500 using three strategies: exhaustive grid search, random search sampling 50 combinations, and Bayesian optimization over 100 iterations; record Silhouette scores per strategy and parameter set. Write the consolidated cluster labels and Silhouette score table to clustered_segmentation_results.csv. MSIght ships as a pip-installable Python package with a Jupyter notebook explaining the pipeline step-by-step, deliberately written in Python to stay accessible to non-experts.

## Related tools

- **MSIght** (Provides the full clustering pipeline: imzML loading/preprocessing, PCA+t-SNE dimensionality reduction, k-means clustering with Silhouette scoring, and hyperparameter optimization modules (refactor_segment, refactor_cluster_optimize)) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language chosen for popularity and accessibility so non-expert users can run the clustering workflow)
- **Jupyter** (Distributable notebook that explains the analysis pipeline (including clustering) step-by-step for programming novices)

## Examples

```
from MSIght.refactor_segment import cluster_msi
```

## Evaluation signals

- A Silhouette score is computed for every segmentation result and reported per strategy (exhaustive, random, Bayesian)
- The best strategy's Silhouette score confirms the Silhouette-optimal t-SNE parameter combination, as visualized in Figure 2A and Supplementary Figures S2–S4
- Cluster count is fixed at k=8 across compared runs so Silhouette values are comparable
- Consolidated cluster labels and Silhouette score table are written to clustered_segmentation_results.csv
- Search bounds are respected: perplexity within 5–50, learning rate within 200–500, iterations within 500–1500; random search samples exactly 50 combinations and Bayesian optimization runs 100 iterations

## Limitations

- Exhaustive, random, and Bayesian optimization yielded similar Silhouette values for MSI segmentation, so expensive exhaustive search may offer little benefit over cheaper strategies
- Default t-SNE settings (perplexity=30, learning_rate=200, n_iter=500) are not guaranteed Silhouette-optimal; without hyperparameter search, segmentation quality can be suboptimal
- Silhouette is the sole validation metric in this workflow; biological/histological concordance should be checked separately, e.g., by expert visual review against co-registered H&E

## Evidence
