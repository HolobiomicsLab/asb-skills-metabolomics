---
name: msms-similarity-network-cluster-visualization
description: Use when exploring LC-MS/MS .mgf feature spectra with specXplore and
  you want cluster groupings (at several k values) of features based on ms2deepscore
  similarities, not edge-threshold-driven partial networks.
when_to_use_negative:
- The goal is edge-level connectivity analysis driven by similarity thresholds and
  top-K neighbor limits, not feature groupings independent of edge filtering.
- The dataset exceeds roughly 5000 features or very liberal settings are used, since
  the paper only anticipates (not tests) scaling beyond that point.
- The spectra fall outside or far from ms2deepscore's training data domain, where
  similarity scores are expected to be unreliable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Dashboard whose t-SNE overview panel hosts the k-medoid cluster color overlay;
    importing pipeline supplies the ms2deepscore similarity matrix.
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores used as
    the quantitative backbone for clustering.
- name: kmedoids
  role: Python package used to run the k-medoid partitioning over the converted similarity
    matrix.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:58d9918611449b3494fa78c66c1de2d3c6573001f973b28f74d00898652e598f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - kmedoids
  - ms2deepscore
  - pandas
  - specXplore
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msms-similarity-network-cluster-visualization

## Summary

Groups MS/MS features by k-medoid clustering over the ms2deepscore pairwise similarity matrix and overlays the resulting cluster labels on the specXplore t-SNE overview, complementing local connectivity views with cluster-level structure.

## When to use

Use when exploring LC-MS/MS .mgf feature spectra with specXplore and you want cluster groupings (at several k values) of features based on ms2deepscore similarities, not edge-threshold-driven partial networks.

## When NOT to use

- The goal is edge-level connectivity analysis driven by similarity thresholds and top-K neighbor limits, not feature groupings independent of edge filtering.
- The dataset exceeds roughly 5000 features or very liberal settings are used, since the paper only anticipates (not tests) scaling beyond that point.
- The spectra fall outside or far from ms2deepscore's training data domain, where similarity scores are expected to be unreliable.

## Inputs

- specXplore session data object from the importing pipeline (mgf MS/MS spectra, feature_id keys, ms2deepscore pairwise similarity matrix)
- Set of k values to test for k-medoid clustering
- Matched feature identifiers for every spectrum

## Outputs

- cluster_assignments.csv with one row per feature and one cluster-label column per tested k
- Cluster color overlay displayed on the specXplore t-SNE overview panel

## How to apply

Run the specXplore importing pipeline on the .mgf MS/MS spectra (matchms import, ms2deepscore/scored spectra with the "feature_id" key) so a session data object with the ms2deepscore pairwise similarity matrix is produced. Convert the pairwise ms2deepscore similarity matrix into the distance representation required for partitioning, then apply k-medoid clustering (kmedoids package) over a user-selected range of k, recording each feature's cluster label at every k; the paper prescribes no automatic tuning guideline for choosing k, and higher k subdivides the space into smaller localized clusters. Assemble one row per feature and one label column per k into a cluster assignment table (cluster_assignments.csv). Use the labels as a color overlay on the t-SNE overview panel and cross-check that the groupings align with topological insights from partial network drawings at the current similarity-threshold and top-K neighbor settings.

## Related tools

- **specXplore** (Dashboard whose t-SNE overview panel hosts the k-medoid cluster color overlay; importing pipeline supplies the ms2deepscore similarity matrix.) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores used as the quantitative backbone for clustering.)
- **kmedoids** (Python package used to run the k-medoid partitioning over the converted similarity matrix.)

## Examples

```
from specxplore.clustering import kmedoids; labels = kmedoids(similarity_matrix, k=10)
```

## Evaluation signals

- cluster_assignments.csv has exactly one row per feature and one label column per tested k, with identifiers matching the ms2deepscore matrix order.
- Assignments are nondegenerate across the tested k range (no single-cluster collapse) and higher k subdivides clusters into smaller localized groups.
- Overlay on the t-SNE overview shows good agreement with t-SNE projections and topological insights from partial network drawings.
- Cluster groupings remain informative and complementary to the t-SNE overview even when the similarity-threshold and top-K edge settings are changed.

## Limitations

- No quantitative optimality criterion or automatic k tuning guideline is provided; more work on measuring optimality is needed.
- Cluster assignments are not enforced to agree across different values of k and may vary across settings.
- Hierarchical clustering with medoid linkage was not used (lacking Python implementation and suboptimal clusters at any level of k).

## Evidence

- [methods] Clustering based on pairwise similarities: "The tool is available on github under MIT license as a python package that can be downloaded and installed for local use"
- [results] k-medoid agreement with t-SNE: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
- [results] k-medoid vs edge filters: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
- [discussion] no automatic k tuning: "More work on measuring and comparing optimality would be needed to provide users with stronger guidelines for automatic cluster tuning."
- [discussion] cluster assignment consistency: "For k-medoid clustering, no such agreement across different values of K is enforced, and hence, cluster assignments may vary across settings of k"
- [discussion] hierarchical medoid linkage not used: "due to both lacking implementation availability in Python and suboptimality of the hierarchically constrained clusters at any level of k"
