---
name: metabolite-feature-selection-by-missingness
description: Use when a QC'd metabolomics feature matrix contains groups of highly
  correlated metabolites and you must reduce to a subset of ~independent features
  (one representative per correlation cluster) while retaining quality-based metadata.
when_to_use_negative:
- The input is already a small, uncorrelated feature panel where no redundancy reduction
  is needed — clustering would arbitrarily drop metabolites.
- You need to select metabolites by a criterion other than least missingness within
  correlated groups (e.g. by biological annotation or platform priority); metaboprep's
  feature_selection parameter would need overriding.
- The dataset has not yet passed sample/feature missingness QC (sample_missingness
  = 0.2, feature_missingness = 0.2) — representatives would be chosen on unreliable
  missingness estimates.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: metaboprep
  role: Hosts the full QC pipeline (quality_control) whose tree_cut_height and feature_selection='least_missingness'
    parameters drive the representative-feature selection and produce the independent_features
    annotation
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment; the clustering uses cutree() from the 'stats' package
    within R
- name: dendextend
  role: Loaded in the article's methods vignette for manipulating/visualising the
    feature dendrogram produced by the clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:3cb328ed7f41505e56e81553bd538856f689d3fc52771eba987fe95285ddde3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - dendextend
  - metaboprep
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-feature-selection-by-missingness

## Summary

Identifies approximately independent (representative) metabolites by hierarchically clustering correlated features and tagging, per cluster, the member with the least missingness — the metaboprep data-reduction step built on Spearman correlation distance, complete linkage, and cutree at a user-defined tree cut height.

## When to use

Use when a QC'd metabolomics feature matrix contains groups of highly correlated metabolites and you must reduce to a subset of ~independent features (one representative per correlation cluster) while retaining quality-based metadata.

## When NOT to use

- The input is already a small, uncorrelated feature panel where no redundancy reduction is needed — clustering would arbitrarily drop metabolites.
- You need to select metabolites by a criterion other than least missingness within correlated groups (e.g. by biological annotation or platform priority); metaboprep's feature_selection parameter would need overriding.
- The dataset has not yet passed sample/feature missingness QC (sample_missingness = 0.2, feature_missingness = 0.2) — representatives would be chosen on unreliable missingness estimates.

## Inputs

- Metaboprep object (or list of data/features/samples) built from a metabolon-style Excel matrix (sheet 'OrigScale') via read_metabolon
- Feature set already filtered at feature_missingness = 0.2 (user-defined threshold)
- QC parameter set including tree_cut_height (0.5) and feature_selection ('least_missingness')

## Outputs

- Feature-level table with feature_id and binary independent_features columns (one flagged representative per dendrogram cluster), e.g. independent_features.csv
- Feature clustering dendrogram (hclust tree stored as 'qc_tree' attribute of feature_summary)

## How to apply

Load the metabolite matrix (e.g. metabolon_v1.1_example.xlsx, sheet 'OrigScale') via read_metabolon into a Metaboprep object, and restrict to features passing the feature_missingness = 0.2 threshold. Compute pairwise Spearman rank correlations among the remaining features and convert to a distance matrix as 1 - |rho|. Build a complete-linkage hierarchical clustering dendrogram on this distance matrix and cut it with cutree() from the 'stats' package at the user-defined tree_cut_height (0.5 in the package examples) to define correlated feature clusters. For each cluster, tag the member with the lowest missingness as the independent representative — the quality_control pipeline parameter feature_selection = "least_missingness" encodes this rule — and emit a binary independent_features column alongside feature_id. Write the resulting feature-level table (e.g. independent_features.csv); the dendrogram itself is retrievable via attr(mydata@feature_summary, "qc_tree") for inspection with dendextend or base plot.

## Related tools

- **metaboprep** (Hosts the full QC pipeline (quality_control) whose tree_cut_height and feature_selection='least_missingness' parameters drive the representative-feature selection and produce the independent_features annotation) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment; the clustering uses cutree() from the 'stats' package within R)
- **dendextend** (Loaded in the article's methods vignette for manipulating/visualising the feature dendrogram produced by the clustering)

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, tree_cut_height=0.5, feature_selection="least_missingness")
```

## Evaluation signals

- Output feature table contains exactly the two key columns: feature_id and a binary independent_features column.
- Each dendrogram cluster formed at tree_cut_height = 0.5 contributes exactly one independent_features-flagged representative (qualitative acceptance check; the paper reports no quantitative metric for this step in isolation).
- Every flagged representative has the lowest missingness among its cluster members (feature_selection = 'least_missingness' invariant).
- Only features passing the 0.2 feature_missingness threshold appear as candidate cluster members in the input matrix.
- The tree retrieved via attr(mydata@feature_summary, "qc_tree") is a complete-linkage dendrogram on 1 - |Spearman rho| distances and can be plotted to eyeball cluster structure.

## Limitations

- The paper reports no quantitative validation metric for the feature-independence step in isolation; acceptance is qualitative (one representative per cluster).
- The tree_cut_height (0.5 in examples) is user-defined, so the number/size of clusters — and hence how aggressively redundancy is removed — depends entirely on that choice.
- metaboprep is marked experimental (lifecycle badge in the README), and the README example shows the pipeline may warn when requested PCs exceed available informative PCs during the coupled PCA outlier step, indicating parameter sensitivity on small datasets.

## Evidence

- [methods] tree_cut_height parameter: "tree_cut_height     = 0.5"
- [methods] feature_selection rule: "feature_selection   = "least_missingness""
- [intro] package goal: filtering pipeline with user thresholds: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [readme] README quality_control invocation with clustering parameters: "mydata <- mydata |> quality_control( source_layer        = "input", 
                                     sample_missingness  = 0.2, 
                                     feature_missingness = 0.2"
- [readme] README access to the feature tree for inspection: [evidence span withheld — anchor status: external_doc]
