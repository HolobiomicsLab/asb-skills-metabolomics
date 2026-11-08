---
name: undirected-graph-connected-component-decomposition
description: Use when a preprocessed LC-MS peak table contains multiple correlated
  features per metabolite (e.g. adducts/isotopes) that must be collapsed into one
  representative per compound before statistics or identification.
when_to_use_negative:
- Input is raw LC-MS instrument data rather than a preprocessed peak table — notame
  expects data already peak-picked (e.g. by MS-DIAL)
- The feature table has already been aggregated so that each metabolite is a single
  unique feature with no adduct/isotope multiplicity left to group
- The goal is clustering of samples (phenotypes) rather than grouping molecular features
  that originate from the same metabolite
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides find_connections, find_clusters and assign_cluster_id, which construct
    the correlation graph (corr >= 0.9, RT window +/-1 s), decompose it into connected
    components, prune low-degree nodes, and assign 'Cluster_x' identifiers based on
    largest median peak area
  repo: https://github.com/hanhineva-lab/notame
- name: notame (archived development repo)
  role: Original implementation of the feature-clustering workflow (antonvsdata/notame),
    retained for historical reference; active development moved to hanhineva-lab/notame
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Language in which the notame package is implemented and in which the MetaboSet
    object, graph construction and clustering are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:700ec874aadcf4275a683b1542ba53f677268accc30922581a4568b2b7d0dd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - liquid chromatography–mass spectrometry
  - notame
  - notame (archived development repo)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# undirected-graph-connected-component-decomposition

## Summary

Groups molecular features originating from the same metabolite in an LC-MS data matrix by building an undirected correlation graph (nodes = features, edges = Pearson correlations within a retention-time window) and recursively extracting clusters from its connected components, as implemented in the notame R package.

## When to use

Use when a preprocessed LC-MS peak table contains multiple correlated features per metabolite (e.g. adducts/isotopes) that must be collapsed into one representative per compound before statistics or identification.

## When NOT to use

- Input is raw LC-MS instrument data rather than a preprocessed peak table — notame expects data already peak-picked (e.g. by MS-DIAL)
- The feature table has already been aggregated so that each metabolite is a single unique feature with no adduct/isotope multiplicity left to group
- The goal is clustering of samples (phenotypes) rather than grouping molecular features that originate from the same metabolite

## Inputs

- MS-DIAL-exported Excel peak table (e.g. sample_data_whole.xlsx) read via read_from_excel
- MetaboSet object holding the feature abundance matrix plus feature metadata (retention times) and sample metadata
- Threshold parameters: Pearson correlation cutoff (0.9), retention-time window (+/-1 s), pruning degree threshold (e.g. 0.8 of maximum possible degree)

## Outputs

- MetaboSet in which every feature carries a non-NA Cluster_ID of the form 'Cluster_x'
- cluster_membership.csv with columns Cluster_ID, representative Feature_ID, and member Feature_IDs

## How to apply

Start from a MetaboSet object built (via construct_metabosets) from an MS-DIAL peak table such as sample_data_whole.xlsx read with read_from_excel. Identify all feature pairs whose Pearson correlation exceeds 0.9 AND whose retention times fall within a +/-1 second window using find_connections, since features from the same compound should co-elute and co-vary. Build an undirected graph with features as nodes and the Pearson correlations as edge weights, and decompose it into connected components with find_clusters. Prune each component by iteratively removing the node with the lowest degree (ties broken by lowest sum of edge weights) until every node reaches the degree threshold (e.g. 0.8 of the maximum possible degree); nodes discarded from a cluster seed new clusters, including singletons. Finally, run assign_cluster_id to label each feature 'Cluster_x', where x is the feature with the largest median peak area in the cluster, and write the membership table (Cluster_ID, representative Feature_ID, member Feature_IDs) to cluster_membership.csv.

## Related tools

- **notame** (Provides find_connections, find_clusters and assign_cluster_id, which construct the correlation graph (corr >= 0.9, RT window +/-1 s), decompose it into connected components, prune low-degree nodes, and assign 'Cluster_x' identifiers based on largest median peak area) — https://github.com/hanhineva-lab/notame
- **notame (archived development repo)** (Original implementation of the feature-clustering workflow (antonvsdata/notame), retained for historical reference; active development moved to hanhineva-lab/notame) — https://github.com/antonvsdata/notame
- **R** (Language in which the notame package is implemented and in which the MetaboSet object, graph construction and clustering are executed)

## Examples

```
conn <- find_connections(set, corr_thresh = 0.9, rt_window = 1); set <- find_clusters(set, conn); set <- assign_cluster_id(set)
```

## Evaluation signals

- Every feature in the output MetaboSet receives a non-NA Cluster_ID matching the pattern 'Cluster_x'
- Within each cluster, the retained representative feature has the largest median peak area among members
- All clustered member features are recorded in cluster_membership.csv alongside their representative
- Every edge used to connect members satisfies both criteria: Pearson correlation >= 0.9 and retention-time difference within +/-1 second
- After pruning, every retained node meets the degree threshold (e.g. 0.8 of the maximum possible degree), and discarded nodes appear as separate clusters or singletons

## Limitations

- The notame package API 'is still quite experimental, and breaking changes are possible'; the original antonvsdata/notame repository is migrated and no longer maintained (development moved to hanhineva-lab/notame)
- The +/-1 second retention-time constraint and 0.9 correlation cutoff are tunable; features of one metabolite eluting with larger RT differences or with weaker correlations across batches may be missed
- Iterative pruning removes the lowest-degree node (ties broken by lowest sum of edge weights), which can split weakly connected components into subclusters or singletons rather than one cluster per metabolite

## Evidence

- [readme] The algorithm for clustering molecular features originating from the same compound is based on MATLAB code written by David Broadhurst, Professor of Data Science & Biostatistics in the School of Science, and director of the Centre for Integrative Metabolomics & Computational Biology at the Edith Covan University.: "The algorithm for clustering molecular features originating from the same compound is based on MATLAB code written by David Broadhurst, Professor of Data Science & Biostatistics in the School of"
- [readme] A novel method for clustering similar molecular features: [evidence span withheld — anchor status: external_doc]
