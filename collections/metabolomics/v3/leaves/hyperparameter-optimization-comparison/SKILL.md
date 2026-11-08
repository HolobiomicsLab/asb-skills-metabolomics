---
name: hyperparameter-optimization-comparison
description: Decision support for choosing an optimization strategy to tune t-SNE
  (or similar embedding) hyperparameters when segmenting mass spectrometry imaging
  data and needing to justify search cost vs. Silhouette-validated segmentation quality.
when_to_use_negative:
- The goal is MSI/LC-MS/MS annotation confidence (coregistration of H&E histology
  and untargeted LC-MS/MS confirmation) rather than segmentation hyperparameter tuning
  — use MSIght's core registration and annotation workflow instead
- Input data are already embedded/clustering-ready feature tables where t-SNE tuning
  is not the bottleneck; search strategies would add cost without changing Silhouette
  outcomes
- The embedding algorithm's hyperparameters differ (e.g., UMAP or PCA-only pipelines);
  the perplexity/learning-rate/n_iter grid defined here is t-SNE-specific
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: MSIght
  role: Python platform providing the preprocessing, PCA/t-SNE dimensionality reduction,
    k-means clustering, and Silhouette-scoring functions used to evaluate each hyperparameter
    strategy
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language in which the exhaustive, random, and Bayesian search
    loops are executed over the t-SNE parameter space
- name: Jupyter
  role: Notebook interface through which the MSIght analysis pipeline (including this
    optimization comparison) is distributed and run step-by-step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:fce90628ed4475be6255b8262484986a022e8564accfd835485a213ac6434925
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hyperparameter-optimization-comparison

## Summary

Benchmarks exhaustive grid, random, and Bayesian search strategies for tuning t-SNE hyperparameters (perplexity, learning rate, number of iterations) in MALDI-MSI segmentation, using k-means Silhouette scores as the objective. Applied in the MSIght pipeline to establish that the search strategy choice has minimal effect on segmentation quality.

## When to use

Decision support for choosing an optimization strategy to tune t-SNE (or similar embedding) hyperparameters when segmenting mass spectrometry imaging data and needing to justify search cost vs. Silhouette-validated segmentation quality.

## When NOT to use

- The goal is MSI/LC-MS/MS annotation confidence (coregistration of H&E histology and untargeted LC-MS/MS confirmation) rather than segmentation hyperparameter tuning — use MSIght's core registration and annotation workflow instead
- Input data are already embedded/clustering-ready feature tables where t-SNE tuning is not the bottleneck; search strategies would add cost without changing Silhouette outcomes
- The embedding algorithm's hyperparameters differ (e.g., UMAP or PCA-only pipelines); the perplexity/learning-rate/n_iter grid defined here is t-SNE-specific

## Inputs

- imzML mass spectrometry imaging data (MALDI-MSI, preprocessed via MSIght load_and_preprocess_imzml)
- Hyperparameter search space: perplexity 5–50, learning rate 200–500, n_iter 500–1500
- Cluster count for k-means scoring (k=8 in the reference workflow)

## Outputs

- Silhouette score per optimization strategy (exhaustive, random, Bayesian) and per parameter set
- Consolidated clustering labels and Silhouette score table written to clustered_segmentation_results.csv
- Optimal t-SNE hyperparameter combination, visualized as Figure 2A and Supplementary Figures S2–S4

## How to apply

Load and preprocess the imzML MSI data with MSIght.refactor_common_functions.load_and_preprocess_imzml and create_intensity_matrix, then apply Gaussian smoothing (sigma=1.0, structuring element size 3) before PCA reduction to 100 components and t-SNE to 2 components (initial settings perplexity=30, learning_rate=200, n_iter=500) via apply_dimensionality_reduction. Define the search space as perplexity 5–50, learning rate 200–500, and iterations 500–1500. Evaluate every candidate by clustering the t-SNE embedding into 8 clusters with k-means (MSIght.refactor_cluster_optimize.kmeans_cluster_msi_scored_w_csv) and recording the Silhouette score per segmentation. Run all three strategies on the same space: an exhaustive grid search over the full grid, a random search sampling 50 parameter combinations, and Bayesian optimization over 100 iterations. Consolidate clustering labels and Silhouette scores per strategy into clustered_segmentation_results.csv, then compare strategy-level Silhouette scores to determine the optimal t-SNE parameter combination (reproducing Figure 2A and Supplementary Figures S2–S4). The finding that all three strategies yielded similar Silhouette values is itself the key rationale: an agent can justify choosing the cheapest strategy when scores converge.

## Related tools

- **MSIght** (Python platform providing the preprocessing, PCA/t-SNE dimensionality reduction, k-means clustering, and Silhouette-scoring functions used to evaluate each hyperparameter strategy) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language in which the exhaustive, random, and Bayesian search loops are executed over the t-SNE parameter space)
- **Jupyter** (Notebook interface through which the MSIght analysis pipeline (including this optimization comparison) is distributed and run step-by-step)

## Examples

```
pip install msight; from MSIght.refactor_segment import cluster_msi  # then run the clustering pipeline to score each t-SNE parameter set with Silhouette
```

## Evaluation signals

- A Silhouette score is computed and reported for each of the three strategies (exhaustive, random, Bayesian), not just for the best run
- The best strategy's Silhouette score confirms a single optimal t-SNE perplexity/learning-rate/iteration combination within the defined ranges (5–50, 200–500, 500–1500)
- The consolidated table clustered_segmentation_results.csv contains per-strategy and per-parameter-set Silhouette scores paired with clustering labels
- Scores should be broadly comparable across strategies (the paper's finding: similar Silhouette values across exhaustive, random, and Bayesian search) — a large discrepancy suggests a scoring or search-space error
- Reproduced results match the layout of Figure 2A and Supplementary Figures S2–S4

## Limitations

- The comparison was performed for t-SNE with k-means (k=8) Silhouette scoring on MSI segmentation; conclusions may not transfer to other embedding or clustering algorithms
- Silhouette is an unsupervised internal metric and does not require histological ground truth, so 'optimal' parameters may not align with anatomically correct segmentation
- MSIght is v0.0.1 (initial release, 9/17/2024); function names such as refactor_cluster_optimize scoring routines may change between versions

## Evidence

- [readme] pip install msight: [evidence span withheld — anchor status: external_doc]
