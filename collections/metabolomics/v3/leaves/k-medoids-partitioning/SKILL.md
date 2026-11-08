---
name: k-medoids-partitioning
description: Use when an LC-MS/MS untargeted metabolomics workflow needs structurally
  similar features grouped together — i.e., a pairwise spectral (dis)similarity distance
  matrix is available and a feature-grouping proxy for downstream feature-set testing
  must be generated.
when_to_use_negative:
- Input is a feature intensity/quantification table rather than a pairwise spectral
  distance matrix — k-medoids here operates on spectral (dis)similarity, not on abundance
  data
- Feature grouping based on retention-time or m/z proximity is desired instead of
  mass-spectral structural similarity — msFeaST's clustering uses spectral similarity
  specifically
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: msFeaST
  role: Runs the k-medoids (PAM-style) clustering on the precomputed spectral distance
    matrix, computes per-K Silhouette scores over a grid of K, and emits the per-feature
    cluster-assignment table used as the grouping proxy in the msFeaST pipeline.
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: 'Consumes the k-medoids clusters downstream: the spectral-similarity-based
    feature groups enable feature group-wise statistical testing via the globaltest
    R package.'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:643cd06b974e03cecbcdfbfc0121fdf6c433ccea6b392bce77917182ebb7c1b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# k-medoids-partitioning

## Summary

Partitions LC-MS/MS features into spectral-similarity-based groups by running k-medoids (PAM) clustering on a precomputed pairwise spectral distance matrix, sweeping K over a grid and selecting the K with the maximal mean Silhouette score.

## When to use

Use when an LC-MS/MS untargeted metabolomics workflow needs structurally similar features grouped together — i.e., a pairwise spectral (dis)similarity distance matrix is available and a feature-grouping proxy for downstream feature-set testing must be generated.

## When NOT to use

- Input is a feature intensity/quantification table rather than a pairwise spectral distance matrix — k-medoids here operates on spectral (dis)similarity, not on abundance data
- Feature grouping based on retention-time or m/z proximity is desired instead of mass-spectral structural similarity — msFeaST's clustering uses spectral similarity specifically

## Inputs

- Precomputed pairwise spectral (dis)similarity distance matrix for all LC-MS/MS features
- Grid of candidate cluster counts K to evaluate

## Outputs

- Silhouette-score grid table: one mean Silhouette score per evaluated K
- Selected K value (grid entry with the highest mean Silhouette score)
- Per-feature cluster-assignment table mapping each feature to its k-medoids cluster at the selected K

## How to apply

Load the precomputed pairwise spectral (dis)similarity distance matrix for all LC-MS/MS features — clustering is run on this precomputed matrix, not on raw spectra or feature intensities. Execute k-medoids (PAM-style) clustering in the msFeaST workflow over a comprehensive grid of candidate cluster counts K, reusing the same distance matrix as both the clustering input and the Silhouette-evaluation input. For each K in the grid, compute the mean Silhouette score of the resulting partition from that distance matrix and record per-K scores in a grid table. Identify the K with the maximal mean Silhouette score as the selected number of clusters; the paper reports no additional numeric Silhouette threshold for this selection. Finally, write the per-feature cluster-assignment table mapping each feature to its k-medoids cluster at the chosen K, so that the clusters serve as a proxy for grouping structurally similar features before feature group-wise statistical testing with globaltest.

## Related tools

- **msFeaST** (Runs the k-medoids (PAM-style) clustering on the precomputed spectral distance matrix, computes per-K Silhouette scores over a grid of K, and emits the per-feature cluster-assignment table used as the grouping proxy in the msFeaST pipeline.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Consumes the k-medoids clusters downstream: the spectral-similarity-based feature groups enable feature group-wise statistical testing via the globaltest R package.)

## Examples

```
conda create --name msfeast_environment python=3.10; conda activate msfeast_environment; jupyter-notebook  # open preprocessing_mushroom_type_comparison.ipynb then msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The Silhouette-score grid table contains exactly one mean Silhouette score per K in the evaluated grid
- The selected K equals the grid entry with the maximal mean Silhouette score
- Every feature in the input distance matrix appears in the per-feature cluster-assignment table with exactly one cluster label at the selected K
- The number of distinct cluster labels in the final assignment table equals the selected K
- Clustering and Silhouette computation both used the same precomputed distance matrix (no recomputation from raw spectra)

## Limitations

- The number of clusters k is selected by maximal mean Silhouette score over a grid of K values; the paper reports no additional numeric threshold for this step, so the chosen K is data-dependent
- msFeaST pre-processing and pipeline were tested on macOS and are expected to work identically on Linux; Windows support was still being worked on
- The pipeline requires a working conda environment with Python 3.10 and R 4.3.3 (globaltest 5.56.0 via Bioconductor 3.18); R path caching issues after conda install can break setup
- Clustering operates on the precomputed spectral distance matrix, so quality depends entirely on how that pairwise (dis)similarity was computed upstream

## Evidence

- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] The current msFeaST pre-processing and pipeline workflow has been tested on macos and should work identically on linux operating systems. Windows support is currently being worked on.: [evidence span withheld — anchor status: external_doc]
- [readme] conda create --name msfeast_environment python=3.10: [evidence span withheld — anchor status: external_doc]
