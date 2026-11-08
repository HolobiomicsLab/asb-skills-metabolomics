---
name: perplexity-hyperparameter-grid-tuning
description: Use when a 2D t-SNE embedding of spectral features is needed in an LC-MS/MS
  untargeted metabolomics pipeline and the pairwise feature distance/similarity matrix
  (e.g. MS2DeepScore) is already available, so perplexity must be chosen objectively
  rather than by default.
when_to_use_negative:
- When no pairwise spectral-similarity/distance matrix is available — the embedding
  must be built on the same distance structure used for feature grouping, so a raw
  feature table without computed distances is insufficient.
- When the goal is exploratory dashboard viewing of pre-processed example results
  rather than generating a new embedding — in that case only the msFeaST_Dashboard_bundle.html
  and dashboard_data.json are needed, no t-SNE tuning.
- When embedding dimensionality other than 2D is required; the pipeline described
  constructs a 2D t-SNE embedding of all features.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msFeaST
  role: Python workflow module that computes the t-SNE embedding of spectral data
    on the distance structure used for clustering, sweeps the perplexity tuning grid,
    and evaluates embeddings via distance preservation scores and plotting
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:a0f0c8bbe234f190f1d1a1950c3a6d492dd6cb0fe934e68dc0c99af909be891f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# perplexity-hyperparameter-grid-tuning

## Summary

Tunes the t-SNE perplexity hyperparameter by sweeping a grid of candidate values on the spectral-similarity distance structure used by msFeaST, scoring each embedding by distance-preservation and selecting the best value for the final 2D feature embedding.

## When to use

Use when a 2D t-SNE embedding of spectral features is needed in an LC-MS/MS untargeted metabolomics pipeline and the pairwise feature distance/similarity matrix (e.g. MS2DeepScore) is already available, so perplexity must be chosen objectively rather than by default.

## When NOT to use

- When no pairwise spectral-similarity/distance matrix is available — the embedding must be built on the same distance structure used for feature grouping, so a raw feature table without computed distances is insufficient.
- When the goal is exploratory dashboard viewing of pre-processed example results rather than generating a new embedding — in that case only the msFeaST_Dashboard_bundle.html and dashboard_data.json are needed, no t-SNE tuning.
- When embedding dimensionality other than 2D is required; the pipeline described constructs a 2D t-SNE embedding of all features.

## Inputs

- Pairwise feature distance/similarity matrix (e.g. MS2DeepScore spectral-similarity matrix) used for clustering
- Grid of candidate perplexity values
- msFeaST preprocessing output: quantification table, metadata table, and spectral data (e.g. from the OMSW_Pleurotus MS2DeepScore notebook example)

## Outputs

- Per-feature 2D t-SNE embedding-coordinate table (CSV)
- Perplexity tuning-grid score table with distance-preservation scores per candidate perplexity value (CSV)

## How to apply

Load the pairwise feature distance/similarity matrix already computed for spectral-similarity-based feature grouping in msFeaST (k-medoids clustering). Sweep a grid of perplexity values, fitting a t-SNE embedding of all features for each value using this distance structure. Score each candidate embedding by its distance-preservation performance, i.e. agreement between original high-dimensional distances and the 2D embedded distances, and record the tuning-grid score table. Select the perplexity with the best distance-preservation score from the grid, then fit the final 2D t-SNE embedding of all features at that selected perplexity. Write the per-feature 2D embedding-coordinate table and the perplexity tuning-grid score table as CSV files for downstream inspection and integration into the msFeaST dashboard.

## Related tools

- **msFeaST** (Python workflow module that computes the t-SNE embedding of spectral data on the distance structure used for clustering, sweeps the perplexity tuning grid, and evaluates embeddings via distance preservation scores and plotting) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The perplexity tuning-grid score table lists a distance-preservation score for every candidate perplexity value in the grid, and the selected perplexity corresponds to the best score.
- The per-feature 2D embedding-coordinate table has one row per feature with exactly two coordinate columns.
- The distance structure used for t-SNE is the same pairwise feature distance/similarity matrix used for spectral-similarity-based k-medoids feature grouping.
- Both output CSVs (embedding coordinates and tuning-grid scores) are produced and can be loaded into the msFeaST interactive dashboard (dashboard_data.json) for inspection.
- The final 2D embedding, when plotted, visually preserves the high-dimensional distance relationships indicated by the best tuning-grid score.

## Limitations

- The perplexity grid sweep and distance-preservation scoring introduce computational cost proportional to the number of candidate perplexity values fitted.
- Distance-preservation of a 2D t-SNE embedding is inherently lossy; the selected perplexity only minimizes, but does not eliminate, distortion of original high-dimensional distances.
- The msFeaST pipeline has been tested on macOS and should work identically on Linux; Windows support is currently being worked on, which may block running the tuning pipeline locally on Windows systems.
- The tuning depends on the quality of the input distance matrix; the pipeline assumes the same distance structure used for clustering is appropriate for embedding.

## Evidence

- [methods] The msFeaST pipeline computes a t-SNE embedding of the spectral data on the same distance structure used for clustering, and the choice of t-SNE embedding is guided by a tuning grid over perplexity values with associated distance preservation scores: [evidence span withheld — anchor status: not_found]
- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The interactive visualization dashboard works regardless of os on desktop browsers (e.g., firefox, chrome, edge, safari): [evidence span withheld — anchor status: external_doc]
- [abstract] Hence, msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and: "Hence, msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and"
