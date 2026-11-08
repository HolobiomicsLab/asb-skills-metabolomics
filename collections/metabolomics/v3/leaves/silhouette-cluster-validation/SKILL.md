---
name: silhouette-cluster-validation
description: Use when a pairwise spectral similarity/distance matrix exists (e.g.
  from MS2DeepScore or MS2/MS similarity) and you must choose how many k-medoids clusters
  to group LC-MS/MS features into, with no prior label or target K.
when_to_use_negative:
- When a prior label or number of feature groups is already fixed by the experimental
  design — the grid sweep over K and Silhouette comparison is unnecessary.
- When the input is already a feature-cluster assignment table rather than a distance
  matrix; Silhouette scoring requires the underlying pairwise spectral distances.
- When data are feature-set testing targets rather than an ungrouped feature set —
  msFeaST downstream globaltest applies to the resulting clusters, not to raw features.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: msFeaST
  role: Hosts the k-medoids (PAM-style) clustering over a K grid on the pairwise spectral
    distance matrix and the Silhouette-score computation, exposed via its Jupyter-notebook
    pipeline.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:6e8cae252c8b9eaae03a3d65faa73d4f53541ec39cc910890756a0b45f749b68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# silhouette-cluster-validation

## Summary

Selects the number of clusters K for k-medoids feature grouping by sweeping a grid of candidate K values over a precomputed spectral (dis)similarity distance matrix and scoring each partition with Silhouette scores, as done in the msFeaST LC-MS/MS untargeted metabolomics workflow.

## When to use

Use when a pairwise spectral similarity/distance matrix exists (e.g. from MS2DeepScore or MS2/MS similarity) and you must choose how many k-medoids clusters to group LC-MS/MS features into, with no prior label or target K.

## When NOT to use

- When a prior label or number of feature groups is already fixed by the experimental design — the grid sweep over K and Silhouette comparison is unnecessary.
- When the input is already a feature-cluster assignment table rather than a distance matrix; Silhouette scoring requires the underlying pairwise spectral distances.
- When data are feature-set testing targets rather than an ungrouped feature set — msFeaST downstream globaltest applies to the resulting clusters, not to raw features.

## Inputs

- Pairwise spectral (dis)similarity distance matrix across all LC-MS/MS features (precomputed in the msFeaST pipeline)
- Grid of candidate cluster counts K to evaluate

## Outputs

- Silhouette-score grid table with one mean Silhouette score per evaluated K
- Selected K value (the grid entry with maximal mean Silhouette score)
- Final per-feature cluster-assignment table mapping each feature to its k-medoids cluster at the selected K

## How to apply

Load the precomputed pairwise spectral (dis)similarity distance matrix for all LC-MS/MS features into the msFeaST pipeline notebook. Run k-medoids (PAM-style) clustering on that distance matrix — not on raw feature intensities — over a grid of candidate cluster counts K, recomputing the partition for each K. For each K, compute the Silhouette score of the resulting per-feature cluster assignment from the same distance matrix. Record the mean Silhouette score per K in a grid table and pick the K with the maximal mean Silhouette score as the final clustering resolution. The paper reports no additional numeric Silhouette cutoff for this decision; the chosen K is simply the grid value with the highest mean Silhouette score.

## Related tools

- **msFeaST** (Hosts the k-medoids (PAM-style) clustering over a K grid on the pairwise spectral distance matrix and the Silhouette-score computation, exposed via its Jupyter-notebook pipeline.) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment; jupyter-notebook  # then run preprocessing_mushroom_type_comparison.ipynb and msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The Silhouette-score grid table contains exactly one mean Silhouette score per candidate K evaluated.
- The selected K equals the grid entry with the highest mean Silhouette score.
- The final per-feature cluster-assignment table covers all features in the input distance matrix and maps each to exactly one k-medoids cluster.
- Silhouette scores are computed from the same pairwise spectral distance matrix used for clustering, not from a separate similarity metric or raw intensity table.

## Limitations

- The paper reports no additional numeric Silhouette threshold or cutoff for accepting a K; the decision is purely a max-over-grid comparison.
- The workflow depends on a reliable precomputed pairwise spectral distance matrix; if similarity scores are noisy or sparse, the Silhouette grid can be misleading.
- msFeaST pre-processing and pipeline steps are tested on macOS and Linux; Windows support is still being worked on, which constrains where the K-selection step can be run.
- Silhouette selection on the grid gives a single K, but structurally similar features may span several resolution levels — the chosen K is only one defensible operating point.

## Evidence

- [methods] Grid-based Silhouette selection supports the k-medoids grouping step: [evidence span withheld — anchor status: not_found]
- [results] Silhouette-driven K choice in workflow step list: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] README - complete example pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] README - OS support caveat: [evidence span withheld — anchor status: external_doc]
