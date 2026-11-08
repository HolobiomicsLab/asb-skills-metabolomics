---
name: tsne-embedding-of-feature-distance-structure
description: Use when you have a pairwise feature distance or similarity matrix (e.g.
  MS2 spectral similarity) and need a 2D visualization/coordinate table that preserves
  the original distance structure for downstream msFeaST dashboard networking.
when_to_use_negative:
- When only a raw feature-quantification table is available and no pairwise spectral-distance/similarity
  matrix has been computed — the embedding must be fit on the same distance structure
  as the k-medoids grouping, not on abundance data.
- When a 3D or higher-dimensional embedding is required; the pipeline here specifically
  produces a 2D t-SNE embedding for the msFeaST dashboard.
- When the goal is clustering rather than visualization — use the k-medoids spectral-similarity
  grouping step instead, and only apply t-SNE afterwards on its distance matrix.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: msFeaST
  role: Provides the pipeline that computes the t-SNE embedding of the spectral data
    on the same distance structure used for clustering, with a perplexity tuning grid
    and distance-preservation scoring to select the embedding
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:66b7b4c036480a990a94210e3258b056dc3b4f38d2cf62308d768c10368dd129
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tsne-embedding-of-feature-distance-structure

## Summary

Construct a 2D t-SNE embedding of all spectral features directly on the same pairwise distance/similarity matrix that msFeaST uses for k-medoids spectral-similarity clustering, tuning perplexity via a distance-preservation score grid so the 2D projection faithfully reflects the mass-spectral similarity structure.

## When to use

Use when you have a pairwise feature distance or similarity matrix (e.g. MS2 spectral similarity) and need a 2D visualization/coordinate table that preserves the original distance structure for downstream msFeaST dashboard networking.

## When NOT to use

- When only a raw feature-quantification table is available and no pairwise spectral-distance/similarity matrix has been computed — the embedding must be fit on the same distance structure as the k-medoids grouping, not on abundance data.
- When a 3D or higher-dimensional embedding is required; the pipeline here specifically produces a 2D t-SNE embedding for the msFeaST dashboard.
- When the goal is clustering rather than visualization — use the k-medoids spectral-similarity grouping step instead, and only apply t-SNE afterwards on its distance matrix.

## Inputs

- Pairwise feature distance/similarity matrix (the same one used for msFeaST k-medoids spectral-similarity feature grouping)
- Perplexity value grid for t-SNE tuning

## Outputs

- Per-feature 2D t-SNE embedding coordinate table (CSV)
- Perplexity tuning-grid score table (CSV)

## How to apply

Load the pairwise feature distance/similarity matrix already used for spectral-similarity-based feature grouping in msFeaST. Sweep a grid of perplexity values, fitting a t-SNE embedding of all features on that distance structure for each candidate, and score each embedding by its distance-preservation performance (agreement between the original high-dimensional distances and the 2D embedded distances). Select the perplexity with the best distance-preservation score, fit the final 2D t-SNE embedding of all features at that value, and write both the per-feature 2D coordinate table and the perplexity tuning-grid score table as CSV. The embedded coordinates feed the msFeaST dashboard's interactive networking of experimental and structural similarity information. This is deliberately run on the same distance structure as the k-medoids clustering so the 2D view is consistent with the clustering used as a proxy for grouping structurally similar features.

## Related tools

- **msFeaST** (Provides the pipeline that computes the t-SNE embedding of the spectral data on the same distance structure used for clustering, with a perplexity tuning grid and distance-preservation scoring to select the embedding) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The selected perplexity must be the grid entry with the best distance-preservation score (agreement between original high-dimensional distances and 2D embedded distances).
- The per-feature coordinate CSV must contain a 2D coordinate pair for every feature present in the input distance matrix (no dropped features).
- The perplexity tuning-grid CSV must contain one score row per candidate perplexity value swept.
- The embedding must be derived from the same distance/similarity matrix used for k-medoids clustering so 2D geometry is consistent with the spectral-similarity cluster structure.
- The workflow must complete within the msFeaST conda environment (python=3.10, R 4.3.3, globaltest 5.56.0) as installed per README; a failure of the R session check (`rscript -e "catch <- lapply(...)"`) implies the pipeline will not run.

## Limitations

- msFeaST pre-processing and pipeline workflow is tested on macOS and should work identically on Linux; Windows support is still being worked on (the interactive dashboard works regardless of OS in desktop browsers).
- The embedding quality is bounded by the quality of the input pairwise distance/similarity matrix; if that matrix does not faithfully capture mass-spectral structural similarity, the 2D t-SNE view and the perplexity tuning will be misleading.
- Distance preservation in 2D is a trade-off — the tuning grid only selects the best available perplexity; no perplexity guarantees exact preservation of all high-dimensional distances.

## Evidence

- [other] The msFeaST pipeline computes a t-SNE embedding of the spectral data on the same distance structure used for clustering, and the choice of t-SNE embedding is guided by a tuning grid over perplexity values with associated distance preservation scores and embedding plotting capabilities for evaluation.: [evidence span withheld — anchor status: not_found]
- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The workflow is run via the msFeaST Jupyter pipeline notebooks, which require a conda environment (python 3.10, R 4.3.3, globaltest 5.56.0).: [evidence span withheld — anchor status: external_doc]
- [readme] The dashboard consumes pre-processed JSON produced by the notebook pipeline and enables interactive exploration of the embedding alongside experimental and structural similarity information.: [evidence span withheld — anchor status: external_doc]
