---
name: silhouette-score-cluster-evaluation
description: Use when selecting unsupervised segmentation parameters (e.g., t-SNE
  perplexity, learning rate, iteration count) or comparing search strategies, and
  an internal cluster-quality metric is needed to rank k-means segmentations of MSI
  pixel data.
when_to_use_negative:
- The goal is peptide identification confirmation or histology co-registration rather
  than segmentation quality tuning — MSIght's core integration workflow (LC-MS/MS
  confirmation, H&E registration) does not require Silhouette scoring.
- Ground-truth spatial annotations are available, making supervised segmentation metrics
  more appropriate than the unsupervised Silhouette score.
- The clustering labels already exist and only need visualization; the skill's value
  is in scoring candidate segmentations, not rendering final images.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MSIght
  role: Python package providing the MSI preprocessing, t-SNE/PCA dimensionality reduction
    (apply_dimensionality_reduction), and scored k-means clustering (kmeans_cluster_msi_scored_w_csv)
    used to generate and Silhouette-score segmentations
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language in which the exhaustive grid, random, and Bayesian
    search loops over t-SNE hyperparameters are executed and Silhouette scores are
    computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:ec0a7bebf90a8f9fff1fec0fd7815568c05760126067ae0a9c3d1e14eec18ac6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# silhouette-score-cluster-evaluation

## Summary

Evaluates MSI segmentation clusterings by computing Silhouette scores for each parameter combination, using the score to compare hyperparameter search strategies (exhaustive grid, random, Bayesian) for tuning t-SNE embeddings of imzML mass spectrometry imaging data.

## When to use

Use when selecting unsupervised segmentation parameters (e.g., t-SNE perplexity, learning rate, iteration count) or comparing search strategies, and an internal cluster-quality metric is needed to rank k-means segmentations of MSI pixel data.

## When NOT to use

- The goal is peptide identification confirmation or histology co-registration rather than segmentation quality tuning — MSIght's core integration workflow (LC-MS/MS confirmation, H&E registration) does not require Silhouette scoring.
- Ground-truth spatial annotations are available, making supervised segmentation metrics more appropriate than the unsupervised Silhouette score.
- The clustering labels already exist and only need visualization; the skill's value is in scoring candidate segmentations, not rendering final images.

## Inputs

- imzML mass spectrometry imaging dataset (pixel-level spectra)
- t-SNE-embedded pixel feature matrix (2 components after PCA-100 reduction)
- k-means cluster labels (8 clusters) for each candidate hyperparameter combination
- Search-space specification: perplexity 5–50, learning rate 200–500, n_iter 500–1500

## Outputs

- Silhouette score per clustering strategy (exhaustive, random, Bayesian) and per parameter set
- clustered_segmentation_results.csv containing consolidated clustering labels and Silhouette scores
- Optimal t-SNE parameter combination and strategy ranking (Figure 2A / Supplementary Figures S2–S4)

## How to apply

Load and preprocess the imzML MSI data with MSIght (load_and_preprocess_imzml, create_intensity_matrix), apply Gaussian smoothing (sigma=1.0, structuring element size 3), then reduce dimensionality via PCA to 100 components followed by t-SNE to 2 components (initial settings perplexity=30, learning_rate=200, n_iter=500). Cluster the t-SNE-embedded pixels into 8 clusters with k-means via MSIght.refactor_cluster_optimize.kmeans_cluster_msi_scored_w_csv and compute the Silhouette score for each segmentation. Search the same parameter space — perplexity 5–50, learning rate 200–500, iterations 500–1500 — with exhaustive grid search, random search (50 sampled combinations), and Bayesian optimization (100 iterations), scoring every candidate segmentation by Silhouette. Consolidate clustering labels and Silhouette scores per strategy into clustered_segmentation_results.csv; the best-scoring combination identifies the optimal t-SNE parameters (Figure 2A, Supplementary Figures S2–S4).

## Related tools

- **MSIght** (Python package providing the MSI preprocessing, t-SNE/PCA dimensionality reduction (apply_dimensionality_reduction), and scored k-means clustering (kmeans_cluster_msi_scored_w_csv) used to generate and Silhouette-score segmentations) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language in which the exhaustive grid, random, and Bayesian search loops over t-SNE hyperparameters are executed and Silhouette scores are computed)

## Examples

```
pip install msight; from MSIght.refactor_segment import cluster_msi
```

## Evaluation signals

- A Silhouette score is reported for each of the three strategies (exhaustive, random, Bayesian) over the identical parameter space (perplexity 5–50, learning rate 200–500, iterations 500–1500).
- The best strategy's Silhouette score identifies a concrete optimal perplexity/learning rate/iteration combination, matching Figure 2A.
- clustered_segmentation_results.csv is produced with both clustering labels and a Silhouette score table.
- Per-combination scores for Supplementary Figures S2–S4 are reproducible from the recorded search results.
- Each segmentation uses a fixed cluster count (8 k-means clusters) so Silhouette values are comparable across strategies.

## Limitations

- The article's comparison found similar Silhouette values across exhaustive, random, and Bayesian search for MSI segmentation, so the metric may not strongly discriminate between search strategies.
- Scores depend on upstream preprocessing choices (Gaussian smoothing sigma=1.0, structuring element size 3, PCA to 100 components); changing these shifts the Silhouette landscape.
- MSIght was at an early release (version 0.0.1, released 9/17/2024), so module APIs may still change.

## Evidence

- [full_text] Comparison of exhaustive, random, and Bayesian t-SNE optimization methods yielded similar Silhouette values for MSI segmentation.: [evidence span withheld — anchor status: not_found]
