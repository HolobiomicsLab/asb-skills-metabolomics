---
name: metabolite-correlation-distance-clustering
description: Use when a metabolomics feature matrix (e.g. Metabolon OrigScale data)
  contains correlated metabolites and you need a reduced set of approximately independent
  representatives prior to downstream analysis.
when_to_use_negative:
- The input is already an independent/reduced feature set with no correlated metabolites
  to collapse
- The goal is sample-level QC (sample missingness, total_peak_area_sd, PCA outliers)
  rather than feature-level redundancy reduction
- Data are not metabolomics feature matrices readable by metaboprep's readers (e.g.
  raw mzML spectra before quantification)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: metaboprep
  role: R package whose quality_control() pipeline computes Spearman correlation distances,
    builds the feature dendrogram, cuts it at tree_cut_height, and tags least-missingness
    representatives
  repo: github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment; cutree() from the 'stats' package cuts the dendrogram
    at the user-defined tree cut height
- name: dendextend
  role: Loaded in the package vignette to support dendrogram manipulation/visualization
    of the feature tree
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:b59cced8cd731ed436b4640ef84b7fa15fc0aaf56d675776262791490e82a6fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dendextend
  - metaboprep
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metabolite-correlation-distance-clustering

## Summary

Identify clusters of highly correlated (redundant) metabolites by hierarchical clustering on a Spearman rank-correlation distance matrix and tag one representative metabolite per cluster, as implemented in the metaboprep R package's data-reduction step.

## When to use

Use when a metabolomics feature matrix (e.g. Metabolon OrigScale data) contains correlated metabolites and you need a reduced set of approximately independent representatives prior to downstream analysis.

## When NOT to use

- The input is already an independent/reduced feature set with no correlated metabolites to collapse
- The goal is sample-level QC (sample missingness, total_peak_area_sd, PCA outliers) rather than feature-level redundancy reduction
- Data are not metabolomics feature matrices readable by metaboprep's readers (e.g. raw mzML spectra before quantification)

## Inputs

- Metabolomics feature matrix (e.g. metabolon_v1.1_example.xlsx, sheet 'OrigScale') loaded via read_metabolon into a Metaboprep object
- Feature missingness statistics computed during quality_control
- User-defined thresholds: feature_missingness (e.g. 0.2) and tree_cut_height (e.g. 0.5)

## Outputs

- Feature-level table independent_features.csv with a feature_id column and a binary independent_features column
- Feature clustering dendrogram (hclust tree) accessible as attr(mydata@feature_summary, 'qc_tree')

## How to apply

Load the metabolite matrix (e.g. metabolon_v1.1_example.xlsx, sheet 'OrigScale') into a Metaboprep object with read_metabolon, then restrict to features passing the user-defined missingness threshold (feature_missingness = 0.2). Compute pairwise Spearman rank correlations among remaining features and convert to a distance matrix as 1 - |rho|, then build a complete-linkage hierarchical clustering dendrogram. Cut the dendrogram at the user-defined tree_cut_height (0.5 in the package examples) using cutree() from the 'stats' package to define correlated feature clusters. For each cluster, tag the metabolite with the least missingness as the representative (feature_selection = 'least_missingness'), emit the binary independent_features column alongside feature_id, and write the feature-level table to independent_features.csv. The rationale is to retain one approximately independent metabolite per correlated group so downstream association testing is not inflated by redundancy.

## Related tools

- **metaboprep** (R package whose quality_control() pipeline computes Spearman correlation distances, builds the feature dendrogram, cuts it at tree_cut_height, and tags least-missingness representatives) — github.com/MRCIEU/metaboprep
- **R** (Runtime environment; cutree() from the 'stats' package cuts the dendrogram at the user-defined tree cut height)
- **dendextend** (Loaded in the package vignette to support dendrogram manipulation/visualization of the feature tree)

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- Output table independent_features.csv contains both a feature_id column and a binary independent_features column
- Each dendrogram cluster contributes exactly one flagged representative metabolite
- Features excluded at feature_missingness = 0.2 do not appear in the independence selection
- The stored qc_tree (attr(mydata@feature_summary, 'qc_tree')) can be plotted as a dendrogram whose cut at tree_cut_height = 0.5 reproduces the cluster count
- Qualitative acceptance only: the paper reports no quantitative metric for this step in isolation

## Limitations

- The paper reports no quantitative validation metric for the representative-selection step in isolation; acceptance is a qualitative structural check
- Only one representative per correlated cluster is retained, discarding correlated metabolites that may carry distinct information
- Cluster granularity depends entirely on the user-defined tree_cut_height (0.5 in examples), with no default guidance on optimal cut height

## Evidence

- [full_text] Clusters of similar metabolites are identified using cutree() from the 'stats' package at a user-defined tree cut height, and for each cluster the metabolite with the least missingness is tagged as the representative metabolite.: [evidence span withheld — anchor status: not_found]
- [methods] Feature selection parameters in the quality_control call: "tree_cut_height     = 0.5"
- [methods] feature_selection parameter set to least_missingness: "feature_selection   = "least_missingness""
- [readme] README quality_control invocation with feature missingness and tree cut height thresholds: "mydata <- mydata |> quality_control( source_layer        = "input", sample_missingness  = 0.2, feature_missingness = 0.2"
- [readme] README shows the feature dendrogram is stored on the feature summary: [evidence span withheld — anchor status: external_doc]
