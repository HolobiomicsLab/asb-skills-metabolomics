---
name: lcms-feature-correlation-clustering
description: Use when a preprocessed LC-MS feature abundance matrix contains multiple
  features per metabolite (adducts, isotopes) that need grouping, or when redundant
  correlated features must be collapsed before statistical analysis.
when_to_use_negative:
- The data are raw LC-MS instrument files that have not yet been peak-picked into
  a feature table — notame expects preprocessed data from a peak-picking tool such
  as MS-DIAL
- The goal is de novo metabolite identification rather than grouping of already-detected
  features into same-metabolite clusters
- Features should be treated as independent units (e.g. when deliberately retaining
  all adduct/isotope signals separately for later analysis)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: notame
  role: R package supplying find_connections (Pearson correlation with retention-time
    constraint), find_clusters (graph construction, connected-component decomposition
    and degree-based pruning), and assign_cluster_id (Cluster_ID assignment by largest
    median peak area)
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime environment for the notame clustering functions; the package requires
    R 3.5.0 or greater
  repo: https://github.com/antonvsdata/notame
- name: liquid chromatography–mass spectrometry
  role: Data-acquisition technique producing the feature abundance matrix that is
    clustered
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:210564efa617fc4291b5fcdcaafaaeed8cabed0cc5d627d61caa6e60130aad59
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
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lcms-feature-correlation-clustering

## Summary

Clusters molecular features in an LC-MS peak table that originate from the same metabolite (e.g. adducts and isotopologues) by building a correlation graph over features and recursively pruning it into connected components. This reduces redundant features and groups signals of the same compound before downstream statistics.

## When to use

Use when a preprocessed LC-MS feature abundance matrix contains multiple features per metabolite (adducts, isotopes) that need grouping, or when redundant correlated features must be collapsed before statistical analysis.

## When NOT to use

- The data are raw LC-MS instrument files that have not yet been peak-picked into a feature table — notame expects preprocessed data from a peak-picking tool such as MS-DIAL
- The goal is de novo metabolite identification rather than grouping of already-detected features into same-metabolite clusters
- Features should be treated as independent units (e.g. when deliberately retaining all adduct/isotope signals separately for later analysis)

## Inputs

- Preprocessed LC-MS peak table (e.g. MS-DIAL Excel export, read via read_from_excel)
- MetaboSet object built with construct_metabosets, containing the feature abundance matrix and feature metadata (m/z, retention time)
- Threshold parameters: Pearson correlation cutoff (0.9), retention-time window (+/-1 s), degree threshold (e.g. 0.8 of maximum possible degree)

## Outputs

- Cluster-membership table written to cluster_membership.csv with columns Cluster_ID, representative Feature_ID, and member Feature_IDs
- Per-feature Cluster_ID annotations of the form 'Cluster_x' added to the MetaboSet feature metadata

## How to apply

Load a peak table (e.g. the MS-DIAL-derived sample_data_whole.xlsx from the notame repo) via read_from_excel and construct a MetaboSet with construct_metabosets. Identify correlated feature pairs with find_connections using Pearson correlation with a correlation-coefficient threshold of 0.9, restricted to a retention-time window of +/-1 second. Build an undirected graph whose nodes are features and edge weights are the Pearson correlations, and decompose it into connected components with find_clusters. Prune each component by iteratively removing the lowest-degree node (ties broken by lowest sum of edge weights) until every node meets a degree threshold such as 0.8 of the maximum possible degree, with discarded nodes forming new clusters, including singletons. Finally, assign each feature a Cluster_ID of the form 'Cluster_x' with assign_cluster_id, where x names the feature with the largest median peak area in the cluster, and write the membership table (Cluster_ID, representative Feature_ID, member Feature_IDs) to cluster_membership.csv.

## Related tools

- **notame** (R package supplying find_connections (Pearson correlation with retention-time constraint), find_clusters (graph construction, connected-component decomposition and degree-based pruning), and assign_cluster_id (Cluster_ID assignment by largest median peak area)) — https://github.com/antonvsdata/notame
- **R** (Runtime environment for the notame clustering functions; the package requires R 3.5.0 or greater) — https://github.com/antonvsdata/notame
- **liquid chromatography–mass spectrometry** (Data-acquisition technique producing the feature abundance matrix that is clustered)

## Examples

```
conns <- find_connections(object, corr_thresh = 0.9, rt_window = 1); clusters <- find_clusters(conns, min_degree = 0.8); object <- assign_cluster_id(clusters, object)
```

## Evaluation signals

- Every feature in the output receives a non-NA Cluster_ID of the form 'Cluster_x' (singletons are valid clusters)
- Within each cluster, the retained representative feature is the one with the largest median peak area among the members
- All clustered member Feature_IDs are recorded in cluster_membership.csv alongside their Cluster_ID and representative Feature_ID
- Clustered members are consistent with the graph: features sharing a Cluster_ID are connected by edges with Pearson correlation >= 0.9 within the +/-1 s retention-time window
- Pruning terminates: after iterative lowest-degree node removal, every remaining node in a cluster meets the degree threshold (e.g. 0.8 of the maximum possible degree)

## Limitations

- The clustering algorithm is based on MATLAB code by David Broadhurst reimplemented in R, and the notame package API is described as 'quite experimental' with breaking changes possible
- The original antonvsdata/notame repository is no longer maintained; active development has moved to hanhineva-lab/notame, so issues and pull requests should go to the new repository
- Retention-time window (1 s) and correlation threshold (0.9) are fixed choices that may need adjustment for datasets with different chromatographic peak widths or signal-to-noise characteristics

## Evidence

- [readme] A novel method for clustering similar molecular features: [evidence span withheld — anchor status: external_doc]
- [readme] Clustering algorithm provenance: "The algorithm for clustering molecular features originating from the same compound is based on MATLAB code written by David Broadhurst"
- [readme] API stability caveat: "The package API is still quite experimental, and breaking changes are possible"
- [readme] Input expectation: preprocessed data: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [intro] Bundled preprocessing scope: "we use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data"
