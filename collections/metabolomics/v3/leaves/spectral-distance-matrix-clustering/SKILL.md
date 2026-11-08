---
name: spectral-distance-matrix-clustering
description: Use when you already have a pairwise spectral similarity/distance matrix
  for untargeted LC-MS/MS features and need to partition features into structural-similarity
  groups with a data-driven choice of K.
when_to_use_negative:
- Input is a raw feature-intensity table rather than a pairwise spectral (dis)similarity
  distance matrix — k-medoids here requires precomputed spectral distances
- You need intensity/differential-abundance clustering of features rather than grouping
  by mass-spectral structural similarity
- Only a single fixed K is required and no data-driven Silhouette-based selection
  of the cluster count is wanted
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Runs k-medoids clustering on the pairwise spectral distance matrix over a
    grid of K and computes Silhouette scores to select the spectral-similarity-based
    feature groupings
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:d636ab030b19a04abfed9cdad07236ae4e11d8e07bfa757f2eed590c7cba045f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-distance-matrix-clustering

## Summary

Cluster LC-MS/MS features into structurally similar groups by running k-medoids clustering directly on a precomputed pairwise spectral (dis)similarity distance matrix, and select the number of clusters K by maximizing the mean Silhouette score over a grid of candidate K values.

## When to use

Use when you already have a pairwise spectral similarity/distance matrix for untargeted LC-MS/MS features and need to partition features into structural-similarity groups with a data-driven choice of K.

## When NOT to use

- Input is a raw feature-intensity table rather than a pairwise spectral (dis)similarity distance matrix — k-medoids here requires precomputed spectral distances
- You need intensity/differential-abundance clustering of features rather than grouping by mass-spectral structural similarity
- Only a single fixed K is required and no data-driven Silhouette-based selection of the cluster count is wanted

## Inputs

- Precomputed pairwise spectral (dis)similarity distance matrix across all LC-MS/MS features
- Candidate grid of cluster counts K to evaluate

## Outputs

- Silhouette-score grid table with one mean Silhouette score per evaluated K
- Per-feature cluster-assignment table mapping each feature to its k-medoids spectral-similarity cluster at the selected K

## How to apply

Load the precomputed pairwise spectral (dis)similarity distance matrix covering all LC-MS/MS features; msFeaST uses this matrix, not raw intensities, as the clustering input. Run k-medoids (PAM-style) clustering over a comprehensive grid of candidate cluster counts K, evaluating each partition on the same distance matrix. For every K in the grid, compute the mean Silhouette score of the resulting partition, tabulate the scores, and select the K with the maximal mean Silhouette score as the final cluster count. Write the final per-feature cluster-assignment table mapping each spectral-similarity group to every feature; these groups then serve as feature sets for downstream group-wise globaltest statistical testing in the msFeaST pipeline. The paper's rationale is that spectral-similarity clusters act as a proxy for grouping structurally similar features, so clustering must operate on spectral distances rather than sample-wise intensity profiles.

## Related tools

- **msFeaST** (Runs k-medoids clustering on the pairwise spectral distance matrix over a grid of K and computes Silhouette scores to select the spectral-similarity-based feature groupings) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment; jupyter-notebook  # open notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb and run the k-medoids + Silhouette grid step
```

## Evaluation signals

- Silhouette-score grid table contains exactly one mean Silhouette score per evaluated K (no missing grid entries)
- The chosen K is the grid value with the highest mean Silhouette score in the table
- The per-feature cluster-assignment table covers every feature in the input distance matrix with a single non-null cluster label
- Cluster assignments are computed from the same spectral distance matrix used for k-medoids (silhouette on the identical distance input)
- Resulting groups serve as feature sets that feed the subsequent group-wise globaltest step in the msFeaST workflow

## Limitations

- The paper reports no additional numeric threshold beyond maximal mean Silhouette score for accepting the selected K
- The msFeaST preprocessing/pipeline workflow was tested on macOS and is expected to work identically on Linux; Windows support is still being worked on
- Windows use requires running commands from the Anaconda Prompt for conda, and R path caching issues can break installation if the terminal is not reopened after setup

## Evidence

- [supplementary] k-medoids clustering on pairwise spectral distance matrix with Silhouette-based K selection: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral-similarity feature grouping via k-medoids: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] Windows support status: [evidence span withheld — anchor status: external_doc]
