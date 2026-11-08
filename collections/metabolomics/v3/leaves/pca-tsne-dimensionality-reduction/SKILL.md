---
name: pca-tsne-dimensionality-reduction
description: Use when the input is an imzML MSI dataset whose high-dimensional pixel
  spectra must be reduced to a 2D embedding (PCA→t-SNE) for clustering-based tissue
  segmentation and hyperparameter comparison.
when_to_use_negative:
- When the pixel data are already low-dimensional (e.g., a 2-3 feature table) — PCA-to-100-components
  would destroy information or be unnecessary.
- When the goal is LC-MS/MS peptide annotation or histology co-registration (MSIght's
  other modalities) rather than MSI pixel embedding for segmentation.
- When a non-Python workflow is required — Cardinal (R-based) or MSiReader cover this
  need without the MSIght Python package.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSIght
  role: Provides the PCA→t-SNE implementation (apply_dimensionality_reduction), imzML
    loading, and clustering/scoring functions used for the embedding and hyperparameter
    optimization
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Language in which the MSIght dimensionality-reduction pipeline and hyperparameter
    search are implemented and distributed
- name: Jupyter
  role: Notebook through which the MSIght analysis pipeline (including dimensionality
    reduction) is executed step-by-step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:56a2a20ad33d6ece23930d8e4a11432f3f6e87eb568a329f50e0042223a6e742
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# pca-tsne-dimensionality-reduction

## Summary

Dimensionality-reduction skill from the MSIght platform: PCA reduces high-dimensional mass spectrometry imaging (MSI) pixel spectra to 100 components, followed by t-SNE embedding into 2 components for downstream k-means segmentation of tissue. Hyperparameters (perplexity, learning rate, n_iter) can be tuned by exhaustive, random, or Bayesian search using Silhouette score.

## When to use

Use when the input is an imzML MSI dataset whose high-dimensional pixel spectra must be reduced to a 2D embedding (PCA→t-SNE) for clustering-based tissue segmentation and hyperparameter comparison.

## When NOT to use

- When the pixel data are already low-dimensional (e.g., a 2-3 feature table) — PCA-to-100-components would destroy information or be unnecessary.
- When the goal is LC-MS/MS peptide annotation or histology co-registration (MSIght's other modalities) rather than MSI pixel embedding for segmentation.
- When a non-Python workflow is required — Cardinal (R-based) or MSiReader cover this need without the MSIght Python package.

## Inputs

- imzML mass spectrometry imaging file (pixel x m/z intensity data)
- Intensity matrix produced by MSIght.refactor_common_functions.create_intensity_matrix
- t-SNE hyperparameter search space: perplexity 5–50, learning rate 200–500, n_iter 500–1500

## Outputs

- 2-component t-SNE embedding of MSI pixels (100 PCA components reduced to 2D)
- Per-strategy Silhouette scores for exhaustive, random, and Bayesian search
- Consolidated clustering labels and Silhouette score table (clustered_segmentation_results.csv)

## How to apply

Load and preprocess the imzML file with MSIght.refactor_common_functions.load_and_preprocess_imzml and create_intensity_matrix, then apply Gaussian smoothing (sigma=1.0, structuring element size 3) to suppress pixel-level noise. Reduce the intensity matrix via PCA to 100 components (retaining most spectral variance while denoising), then embed those components into 2 dimensions with t-SNE using initial parameters perplexity=30, learning_rate=200, n_iter=500 via MSIght.refactor_common_functions.apply_dimensionality_reduction. Feed the 2D embedding into k-means (e.g., 8 clusters) and score each segmentation with Silhouette score; optimize perplexity (5–50), learning rate (200–500), and iterations (500–1500) via exhaustive grid, random (50 samples), or Bayesian (100 iterations) search, comparing strategies by Silhouette score.

## Related tools

- **MSIght** (Provides the PCA→t-SNE implementation (apply_dimensionality_reduction), imzML loading, and clustering/scoring functions used for the embedding and hyperparameter optimization) — https://github.com/lingjunli-research/MSIght
- **Python** (Language in which the MSIght dimensionality-reduction pipeline and hyperparameter search are implemented and distributed)
- **Jupyter** (Notebook through which the MSIght analysis pipeline (including dimensionality reduction) is executed step-by-step)

## Examples

```
from MSIght.refactor_common_functions import load_and_preprocess_imzml, apply_dimensionality_reduction; X = load_and_preprocess_imzml('data.imzml'); X2d = apply_dimensionality_reduction(X, pca_components=100, tsne_components=2, perplexity=30, learning_rate=200, n_iter=500)
```

## Evaluation signals

- PCA reduces the pixel x m/z intensity matrix to exactly 100 components before t-SNE is applied; the final embedding has exactly 2 components.
- Initial t-SNE run uses perplexity=30, learning_rate=200, n_iter=500; any tuned run falls within perplexity 5–50, learning rate 200–500, iterations 500–1500.
- Silhouette score is computed for every segmentation result (each hyperparameter combination and each search strategy: exhaustive, random, Bayesian).
- Exhaustive, random, and Bayesian search yield comparable Silhouette values (the paper's finding), so a large divergence between strategies signals an implementation error.
- The consolidated clustering labels and Silhouette table are written to clustered_segmentation_results.csv, reproducing Figure 2A and Supplementary Figures S2–S4.

## Limitations

- t-SNE is stochastic: embeddings (and thus k-means cluster assignments and Silhouette scores) can vary between runs, so hyperparameter comparisons across search strategies may show noise-level differences.
- The exhaustive grid over perplexity (5–50) x learning rate (200–500) x iterations (500–1500) is computationally expensive; random (50 samples) and Bayesian (100 iterations) searches are the recommended cheaper alternatives.
- Silhouette-optimized hyperparameters for MSI segmentation may not transfer to other datasets; the skill was validated on the paper's imzML data with k=8 k-means clusters.

## Evidence
