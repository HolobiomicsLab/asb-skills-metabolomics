---
name: hierarchical-dendrogram-height-cutting
description: Use when a metabolomics feature matrix contains highly correlated metabolites
  and you need a reduced set of approximately independent representative features
  via hierarchical clustering with a fixed tree-cut height.
when_to_use_negative:
- The feature set is already confirmed to be independent (no correlated clusters),
  so pruning would discard informative metabolites without benefit.
- You need a biologically driven feature grouping rather than a statistical correlation-based
  partition.
- Working outside the metaboprep QC pipeline on data where feature missingness has
  not been quantified, since representative selection depends on per-feature missingness.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: R package whose quality_control() pipeline computes the feature dendrogram,
    applies tree_cut_height and feature_selection, and emits the independent_features
    annotation
  repo: github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment; stats::cutree() performs the height-based dendrogram
    cut
- name: dendextend
  role: Loaded for dendrogram visualization/manipulation of the feature clustering
    tree
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:2ab0ee9ccf3ef71cb09628ce1cdc3880d92c67260ba0896ca15083fce0bfd3ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dendextend
  - metaboprep
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# hierarchical-dendrogram-height-cutting

## Summary

Cuts a hierarchical clustering dendrogram of metabolite features at a user-defined tree height to define clusters of correlated features, then selects one representative metabolite per cluster (the one with least missingness) to form an approximately independent feature subset for downstream analysis.

## When to use

Use when a metabolomics feature matrix contains highly correlated metabolites and you need a reduced set of approximately independent representative features via hierarchical clustering with a fixed tree-cut height.

## When NOT to use

- The feature set is already confirmed to be independent (no correlated clusters), so pruning would discard informative metabolites without benefit.
- You need a biologically driven feature grouping rather than a statistical correlation-based partition.
- Working outside the metaboprep QC pipeline on data where feature missingness has not been quantified, since representative selection depends on per-feature missingness.

## Inputs

- Metaboprep object holding the quality-passing metabolite feature matrix (e.g. read from metabolon_v1.1_example.xlsx, sheet 'OrigScale')
- User-defined tree_cut_height (0.5 in package examples)
- User-defined feature_selection method ('least_missingness')
- Pairwise Spearman correlation-based distance matrix (1 - |rho|) over retained features

## Outputs

- Feature-level table (independent_features.csv) with feature_id and binary independent_features columns, one representative flagged per dendrogram cluster
- Feature clustering dendrogram (qc_tree attribute of feature_summary) for plotting/inspection

## How to apply

Build the dendrogram from pairwise Spearman rank correlations converted to distances as 1 - |rho|, clustered with complete linkage. Cut the resulting tree at the user-defined height (tree_cut_height = 0.5 in metaboprep's examples) using cutree() from the 'stats' package to define correlated feature clusters. Within each cluster, tag the metabolite with the least missingness as the independent representative (feature_selection = 'least_missingness') and emit a binary independent_features column alongside feature_id. This step runs after features passing the 20% missingness threshold (feature_missingness = 0.2) have been retained, so representatives are chosen only among quality-passing features. The cut is applied inside the quality_control() pipeline and the resulting tree is also retrievable from the feature summary for plotting (e.g. with dendextend).

## Related tools

- **metaboprep** (R package whose quality_control() pipeline computes the feature dendrogram, applies tree_cut_height and feature_selection, and emits the independent_features annotation) — github.com/MRCIEU/metaboprep
- **R** (Runtime environment; stats::cutree() performs the height-based dendrogram cut)
- **dendextend** (Loaded for dendrogram visualization/manipulation of the feature clustering tree)

## Examples

```
mydata <- mydata |> quality_control(source_layer = "input", feature_missingness = 0.2, tree_cut_height = 0.5, feature_selection = "least_missingness")
```

## Evaluation signals

- The output feature table contains a feature_id column and a binary independent_features column.
- Each dendrogram cluster defined by the tree_cut_height cut contributes exactly one flagged representative feature.
- The representative chosen within each cluster is the member with the lowest missingness (feature_selection = 'least_missingness').
- Only features passing the feature_missingness = 0.2 threshold are eligible as representatives.
- The qc_tree attribute retrieved via attr(mydata@feature_summary, "qc_tree") can be plotted and its cluster membership matches the flagged representatives.

## Limitations

- The paper reports no quantitative metric for this step in isolation; validation is a qualitative acceptance check of the output schema and one-representative-per-cluster invariant.
- The tree_cut_height (0.5 in examples) is user-defined, so results are sensitive to the chosen cut height and no default justification is given.
- metaboprep is marked experimental (lifecycle badge), so pipeline behavior may change between versions.

## Evidence

- [methods] Clusters of similar metabolites are identified using cutree() from the 'stats' package at a user-defined tree cut height: [evidence span withheld — anchor status: not_found]
- [methods] tree_cut_height = 0.5: "tree_cut_height     = 0.5"
- [methods] feature_selection   = "least_missingness": "feature_selection   = "least_missingness""
- [readme] Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds.: "Perform data filtering on the data set using a standard pipeline and
according to user-defined thresholds."
- [readme] feature_missingness = 0.2: "feature_missingness = 0.2"
