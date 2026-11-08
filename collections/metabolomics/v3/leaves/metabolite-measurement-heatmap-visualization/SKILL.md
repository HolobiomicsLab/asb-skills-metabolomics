---
name: metabolite-measurement-heatmap-visualization
description: Use when a MultiAssayExperiment of raw/imputed metabolite measurements
  needs QC inspection for outlier samples prior to normalization, especially when
  samples group by a trait like tumor vs normal.
when_to_use_negative:
- Samples have already been removed and variance stabilizing normalization applied
  — proceed to downstream correlation-network or differential analysis instead
- The goal is differential testing between groups (Student's T-Test / ANOVA with Benjamini-Hochberg
  correction), not sample QC for outliers
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDiff
  role: R package providing outlier_heatmap and remove_cluster for outlier detection
    and cluster removal
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: runtime environment; MetaboDiff requires R version 4.0.2 or higher
- name: MultiAssayExperiment
  role: container class holding the raw and imputed metabolite measurement slots passed
    to outlier_heatmap
- name: knn_impute
  role: k-nearest-neighbor imputation (cutoff 0.4) producing the imputed measurements
    visualized in the heatmap
  repo: http://github.com/andreasmock/MetaboDiff/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:26aad185614c634bbe1c2c4440bfa7c5106b26fc88a30fff09a0a5c3e34b5898
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - MultiAssayExperiment
  - R
  - knn_impute
  - outlier_heatmap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolite-measurement-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes relative metabolite measurements as an outlier heatmap with hierarchical clustering plus k-means annotation in the MetaboDiff R package, so that discrepant sample clusters (e.g. a 5-sample cluster with lower measurements) can be identified and excluded via remove_cluster before normalization.

## When to use

Use when a MultiAssayExperiment of raw/imputed metabolite measurements needs QC inspection for outlier samples prior to normalization, especially when samples group by a trait like tumor vs normal.

## When NOT to use

- Samples have already been removed and variance stabilizing normalization applied — proceed to downstream correlation-network or differential analysis instead
- The goal is differential testing between groups (Student's T-Test / ANOVA with Benjamini-Hochberg correction), not sample QC for outliers

## Inputs

- MultiAssayExperiment object with 'raw' and 'imputed' metabolite measurement slots (e.g. met_example from MetaboDiff)
- group_factor character naming the sample trait column (e.g. 'tumor_normal')
- label_colors vector of per-group colors
- k, the number of clusters for the k-means annotation (e.g. k=2)

## Outputs

- Outlier heatmap plot (saved e.g. as outlier_heatmap.png) with hierarchical clustering and k-means cluster annotation
- Filtered MultiAssayExperiment returned by remove_cluster with the outlier cluster's samples dropped from 'raw' and 'imputed'

## How to apply

Load the imputed met_example MultiAssayExperiment built by create_mae and knn_impute (missing-value cutoff 0.4), then run outlier_heatmap on the imputed measurements, setting group_factor to the phenotype of interest (e.g. 'tumor_normal'), label_colors to the two-group color vector (e.g. c('darkseagreen','dodgerblue')), and k=2 so the k-means clustering is drawn as annotation alongside the hierarchical clustering. Inspect which k-means cluster contains samples with on average lower relative metabolite measurements. Rationale: combining hierarchical and k-means views makes sample-to-sample discordance visible before variance stabilizing normalization (vsn). Once a discrepant cluster is confirmed, apply remove_cluster(mae, cluster=<id>) to drop those samples from the 'raw' and 'imputed' slots, then proceed to the Normalization step.

## Related tools

- **MetaboDiff** (R package providing outlier_heatmap and remove_cluster for outlier detection and cluster removal) — http://github.com/andreasmock/MetaboDiff/
- **R** (runtime environment; MetaboDiff requires R version 4.0.2 or higher)
- **MultiAssayExperiment** (container class holding the raw and imputed metabolite measurement slots passed to outlier_heatmap)
- **knn_impute** (k-nearest-neighbor imputation (cutoff 0.4) producing the imputed measurements visualized in the heatmap) — http://github.com/andreasmock/MetaboDiff/

## Examples

```
met_example <- outlier_heatmap(met_example, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue'), k=2); met_example <- remove_cluster(met_example, cluster=1)
```

## Evaluation signals

- The heatmap reproduces the tutorial figure showing a distinct 5-sample cluster (k-means cluster 1) with on average lower relative metabolite measurements
- summary() of the object returned by remove_cluster(met_example, cluster=1) confirms 5 samples were removed
- The 'raw' and 'imputed' slots of the returned object have identical (reduced) sample dimensions
- The k-means annotation in the heatmap uses k=2 clusters and the specified label_colors for the group_factor trait

## Limitations

- The number of k-means clusters must be chosen by the user (the tutorial exemplary data set k=2); a poor k can obscure or split outlier clusters
- Cluster numbering is not guaranteed to be stable across runs, so the cluster id passed to remove_cluster must be re-verified on the produced heatmap
- Requires an imputed object — missing values should first be handled with knn_impute at the recommended 40% (0.4) cutoff

## Evidence

- [methods] outlier heatmap with k-means annotation: "the results of a k-means clustering. In the examplary data we set 2 clusters (k=2"
- [methods] cluster removal step: "we apply the function `remove_cluster` to remove cluster 1"
