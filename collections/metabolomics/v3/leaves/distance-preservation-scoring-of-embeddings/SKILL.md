---
name: distance-preservation-scoring-of-embeddings
description: Use when a 2D t-SNE embedding of spectral or feature-distance data must
  be tuned — choose a perplexity from a grid by comparing each embedding's distance-preservation
  performance rather than picking a default value.
when_to_use_negative:
- The input is already a 2D embedding or a low-dimensional coordinate table — no distance-preservation
  tuning is needed.
- The pairwise distance structure is not shared with the clustering (e.g. the embedding
  is computed on raw intensities rather than the spectral-similarity distance matrix),
  so distance-preservation scores would not reflect the clustering-relevant structure.
- Only a single perplexity is required without evaluation — e.g. the embedding is
  purely illustrative and no perplexity tuning grid is part of the workflow.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Provides the t-SNE embedding of spectral data on the pairwise distance structure
    used for clustering, plus the perplexity tuning grid with associated distance-preservation
    scores and plotting for evaluation
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:a653bf67280e85fc9a40405a18c0108067c051592b5c1251760e9cdd543fa32a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# distance-preservation-scoring-of-embeddings

## Summary

Sweep a grid of t-SNE perplexity values and score each candidate 2D embedding by how well it preserves the high-dimensional pairwise distance structure (e.g. the mass-spectral feature distance matrix used in msFeaST), then select the perplexity with the best distance-preservation score.

## When to use

Use when a 2D t-SNE embedding of spectral or feature-distance data must be tuned — choose a perplexity from a grid by comparing each embedding's distance-preservation performance rather than picking a default value.

## When NOT to use

- The input is already a 2D embedding or a low-dimensional coordinate table — no distance-preservation tuning is needed.
- The pairwise distance structure is not shared with the clustering (e.g. the embedding is computed on raw intensities rather than the spectral-similarity distance matrix), so distance-preservation scores would not reflect the clustering-relevant structure.
- Only a single perplexity is required without evaluation — e.g. the embedding is purely illustrative and no perplexity tuning grid is part of the workflow.

## Inputs

- Pairwise feature distance/similarity matrix used for spectral-similarity-based feature grouping in msFeaST
- Spectral data (LC-MS/MS features) whose 2D embedding is being constructed
- Grid of t-SNE perplexity values to test

## Outputs

- Per-feature 2D t-SNE embedding-coordinate table (CSV)
- Perplexity tuning-grid score table (CSV), mapping each perplexity value to its distance-preservation score
- Selected perplexity value with the best distance-preservation score

## How to apply

Load the pairwise feature distance/similarity matrix that msFeaST already uses for spectral-similarity-based feature grouping (k-medoids clustering). Sweep a grid of t-SNE perplexity values, fitting a t-SNE embedding of all features for each value on that same distance structure. Score each candidate embedding by its distance-preservation performance — the agreement between the original high-dimensional pairwise distances and the distances in the 2D embedding. Select the perplexity whose embedding achieves the best distance-preservation score from the tuning grid, fit the final 2D t-SNE embedding at that perplexity, and write the per-feature 2D coordinate table plus the perplexity-vs-score tuning-grid table as CSV files for downstream use in the msFeaST interactive dashboard.

## Related tools

- **msFeaST** (Provides the t-SNE embedding of spectral data on the pairwise distance structure used for clustering, plus the perplexity tuning grid with associated distance-preservation scores and plotting for evaluation) — https://github.com/kevinmildau/msFeaST

## Examples

```
python -m msfeast_pipeline --input spectral_features.csv --distance-matrix feature_distances.npy --tsne-perplexity-grid 5,10,30,50 --output-embedding features_2d.csv --output-scores perplexity_scores.csv
```

## Evaluation signals

- The selected perplexity value is the one whose embedding has the highest distance-preservation score across the full tuning grid.
- The output includes both a per-feature 2D coordinate table and a perplexity-vs-score tuning-grid table, both written as CSV files.
- The 2D embedded pairwise distances agree well with the original high-dimensional pairwise distances for the selected perplexity (quantified by the distance-preservation score).
- The embedding is computed on the same distance structure used for the k-medoids feature grouping, so the resulting clusters and the embedding are consistent.
- The final t-SNE embedding covers all features in the spectral dataset, not a subset.

## Limitations

- The tuning grid requires fitting a separate t-SNE embedding for every perplexity value, which scales linearly with grid size and can be computationally expensive for large feature counts.
- Distance-preservation scores are only meaningful when the embedding is computed on the same pairwise distance structure used for the downstream clustering; using a different distance measure invalidates the comparison.
- t-SNE is stochastic, so repeated fits at the same perplexity can yield different embeddings and slightly different scores; the selected perplexity should be reported alongside its score rather than assumed reproducible.

## Evidence

- [results] The msFeaST pipeline computes a t-SNE embedding of the spectral data on the same distance structure used for clustering, and the choice of t-SNE embedding is guided by a tuning grid over perplexity values with associated distance preservation scores: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral similarity-based feature grouping via k-medoids clustering: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The msFeaST dashboard and pipeline outputs are produced by a Jupyter notebook that also produces the text file in JSON format for interactive exploration: [evidence span withheld — anchor status: external_doc]
