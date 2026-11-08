---
name: feature-cluster-assignment-tabulation
description: 'Use when untargeted LC-MS/MS feature tables need grouping by structural
  similarity: a precomputed pairwise spectral distance matrix is available and the
  number of clusters k must be chosen from Silhouette-score evaluation.'
when_to_use_negative:
- When the input is already a feature table lacking a pairwise spectral distance matrix
  — msFeaST requires the precomputed distance matrix, not raw peak intensities
- When the goal is feature-wise (per-feature) statistical testing rather than grouping
  — use the globaltest skill for feature-set testing instead
- When a priori biological knowledge already defines structurally similar feature
  groups and no data-driven clustering is needed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: msFeaST
  role: Runs the k-medoids clustering over the K grid, computes Silhouette scores,
    and emits the per-feature cluster-assignment table
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:2591eeb6385370880e27ddfbe35e08be7fbedbe29df31fdff5bfb7a2754ad60a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# feature-cluster-assignment-tabulation

## Summary

Outputs a per-feature cluster-assignment table after k-medoids clustering on the pairwise spectral distance matrix, selecting k via mean Silhouette scores across a grid of K values; enables msFeaST's spectral similarity-based feature grouping.

## When to use

Use when untargeted LC-MS/MS feature tables need grouping by structural similarity: a precomputed pairwise spectral distance matrix is available and the number of clusters k must be chosen from Silhouette-score evaluation.

## When NOT to use

- When the input is already a feature table lacking a pairwise spectral distance matrix — msFeaST requires the precomputed distance matrix, not raw peak intensities
- When the goal is feature-wise (per-feature) statistical testing rather than grouping — use the globaltest skill for feature-set testing instead
- When a priori biological knowledge already defines structurally similar feature groups and no data-driven clustering is needed

## Inputs

- Precomputed pairwise spectral (dis)similarity distance matrix for all LC-MS/MS features
- Grid of candidate cluster counts K to evaluate

## Outputs

- Silhouette-score grid table with one mean Silhouette score per evaluated K
- Final per-feature cluster-assignment table mapping each feature to its k-medoids cluster at the selected K

## How to apply

Load the precomputed pairwise spectral (dis)similarity distance matrix for all LC-MS/MS features. Run k-medoids (PAM-style) clustering in the msFeaST workflow over a grid of candidate cluster counts K, supplying the precomputed distance matrix as input. For each K, compute the mean Silhouette score from the same distance matrix and record it in a grid table. Select the K with the maximal mean Silhouette score as the final cluster count. Write the final per-feature cluster-assignment table mapping each feature to its k-medoids cluster at the selected K. The paper reports no additional numeric threshold beyond maximal Silhouette for choosing K, so the decision point is the grid maximum rather than a fixed cutoff.

## Related tools

- **msFeaST** (Runs the k-medoids clustering over the K grid, computes Silhouette scores, and emits the per-feature cluster-assignment table) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The Silhouette-score grid table contains one mean Silhouette score per evaluated K
- The chosen K equals the grid value with the highest mean Silhouette score
- The cluster-assignment table has one row per LC-MS/MS feature and a valid cluster label for every feature
- Cluster labels are drawn from the same distance matrix used for clustering, ensuring internal consistency between the Silhouette computation and the final assignments

## Limitations

- No additional numeric Silhouette threshold is prescribed beyond maximal score, so K selection rests entirely on the grid maximum
- The k-medoids/Silhouette workflow is tied to a precomputed distance matrix; users must supply their own pairwise spectral distances
- msFeaST was tested on macOS and Linux; Windows support is still being worked on, and R path caching issues in conda can break the rscript-based dependency install

## Evidence

- [results] How does the msFeaST pipeline subdivide features into spectral similarity-based groups, and how is the number of clusters k selected?: [evidence span withheld — anchor status: not_found]
- [results] msFeaST performs k-medoids clustering on the pairwise spectral distance matrix, and the user selects the number of clusters k based on Silhouette scores computed over a comprehensive grid of K values.: [evidence span withheld — anchor status: not_found]
- [results] 1. Load the precomputed pairwise spectral (dis)similarity distance matrix for all features. 2. Run k-medoids (PAM-style) clustering in the msFeaST workflow over a grid of candidate cluster counts K, using the precomputed distance matrix as input.: [evidence span withheld — anchor status: not_found]
- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The current msFeaST pre-processing and pipeline workflow has been tested on macos and should work identically on linux operating systems. Windows support is currently being worked on.: [evidence span withheld — anchor status: external_doc]
