---
name: degree-threshold-cluster-pruning
description: Use when a preprocessed LC-MS peak table contains multiple molecular
  features suspected to derive from the same metabolite (adducts, fragments) and they
  must be grouped via correlation-based graph clustering with degree-threshold pruning.
when_to_use_negative:
- Input is raw LC-MS instrument data rather than a peak table — notame expects data
  already peak-picked by software such as MS-DIAL before R preprocessing
- The goal is drift correction, missing-value imputation, or batch effect correction
  (use correct_drift, impute_rf, normalize_batches instead)
- Features already carry cluster annotations and only need flagging or quality filtering
  (flag_detection, flag_quality, flag_contaminants)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: notame
  role: 'Supplies the clustering machinery: find_connections (Pearson correlation
    + RT window), find_clusters (graph decomposition + degree-threshold pruning),
    and assign_cluster_id (Cluster_x labeling by median peak area)'
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime environment for notame; requires R version 3.5.0 or greater
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:0cd51329bd52c2cb06c860e93092cc04ce87b25b33fe3d5a7c2b1346bc5c9c6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - liquid chromatography–mass spectrometry
  - notame
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# degree-threshold-cluster-pruning

## Summary

notame's graph-based method for clustering LC-MS molecular features that originate from the same metabolite: correlated feature pairs form an undirected graph, connected components are pruned by iteratively removing lowest-degree nodes until a degree threshold is met, and each cluster is labeled by its most abundant member feature.

## When to use

Use when a preprocessed LC-MS peak table contains multiple molecular features suspected to derive from the same metabolite (adducts, fragments) and they must be grouped via correlation-based graph clustering with degree-threshold pruning.

## When NOT to use

- Input is raw LC-MS instrument data rather than a peak table — notame expects data already peak-picked by software such as MS-DIAL before R preprocessing
- The goal is drift correction, missing-value imputation, or batch effect correction (use correct_drift, impute_rf, normalize_batches instead)
- Features already carry cluster annotations and only need flagging or quality filtering (flag_detection, flag_quality, flag_contaminants)

## Inputs

- MetaboSet object (or feature abundance matrix with retention times) built from a preprocessed MS-DIAL Excel peak table (e.g. sample_data_whole.xlsx)
- Pearson correlation coefficient threshold (e.g. 0.9)
- Retention-time window constraint (e.g. +/-1 second)
- Degree threshold for pruning (e.g. 0.8 of maximum possible degree)

## Outputs

- Per-feature Cluster_ID assignments of the form 'Cluster_x'
- Cluster-membership table with Cluster_ID, representative Feature_ID (largest median peak area), and member Feature_IDs (e.g. cluster_membership.csv)

## How to apply

Start from a MetaboSet built (via construct_metabosets) from an MS-DIAL peak table read with read_from_excel, e.g. sample_data_whole.xlsx from the notame GitHub repo. Identify feature pairs using Pearson correlation with find_connections, using a correlation coefficient threshold of 0.9 subject to a retention-time constraint of +/-1 second. Build an undirected graph with features as nodes and Pearson correlations as edge weights, then decompose it into connected components with find_clusters. Prune each component by iteratively removing the node with the lowest degree (ties broken by lowest sum of edge weights) until every node reaches the degree threshold (e.g. 0.8 of the maximum possible degree); discarded nodes seed new clusters, including singletons. Finally, assign_cluster_id labels each feature 'Cluster_x', where x indexes the feature with the largest median peak area in the cluster, and the membership table is exported (e.g. cluster_membership.csv).

## Related tools

- **notame** (Supplies the clustering machinery: find_connections (Pearson correlation + RT window), find_clusters (graph decomposition + degree-threshold pruning), and assign_cluster_id (Cluster_x labeling by median peak area)) — https://github.com/antonvsdata/notame
- **R** (Runtime environment for notame; requires R version 3.5.0 or greater)

## Examples

```
Rscript -e 'library(notame); conn <- find_connections(set); clusters <- find_clusters(conn); set <- assign_cluster_id(set, clusters)'
```

## Evaluation signals

- Every feature receives a non-NA Cluster_ID of the form 'Cluster_x'
- Within each cluster, the retained representative feature has the largest median peak area of all members
- All clustered member features are recorded in the exported cluster-membership table
- After pruning, every retained node in each component meets the degree threshold (e.g. 0.8 of maximum possible degree), and discarded nodes reappear as members of new clusters, including singleton clusters
- Edges in the graph correspond to feature pairs with Pearson correlation above the threshold (e.g. 0.9) whose retention times fall within the +/-1 second window

## Limitations

- The clustering algorithm is a port of MATLAB code by David Broadhurst (Edith Cowan University); behavior should be validated against the original implementation
- The notame package API is described as 'still quite experimental' and breaking changes are possible
- The original repository (antonvsdata/notame) is archived and no longer maintained; active development moved to hanhineva-lab/notame

## Evidence

- [readme] A novel method for clustering similar molecular features: [evidence span withheld — anchor status: external_doc]
- [readme] The algorithm for clustering molecular features originating from the same compound is based on MATLAB code written by David Broadhurst: "The algorithm for clustering molecular features originating from the same compound is based on MATLAB code written by David Broadhurst"
- [intro] we use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data: "we use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data"
