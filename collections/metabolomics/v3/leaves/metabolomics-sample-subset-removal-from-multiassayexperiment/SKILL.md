---
name: metabolomics-sample-subset-removal-from-multiassayexperiment
description: Use when a metabolomics MultiAssayExperiment object (post kNN imputation)
  shows a distinct outlier cluster in sample-sample clustering that should be excluded
  before normalization.
when_to_use_negative:
- The metabolite measurement table has not yet been merged into a MultiAssayExperiment
  object and imputed (knn_impute with 0.4 cutoff) — outlier_heatmap operates on the
  imputed measurements
- No distinct outlier cluster is apparent and removal would discard biologically meaningful
  samples without visual clustering evidence
- The goal is group-level differential testing (T-Test/ANOVA with Benjamini-Hochberg
  correction) rather than pre-normalization sample quality control
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDiff
  role: R package providing outlier_heatmap and remove_cluster for outlier identification
    and cluster removal from the MultiAssayExperiment object
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment (R >= 4.0.2) in which outlier_heatmap and remove_cluster
    are executed
- name: MultiAssayExperiment
  role: Data container whose 'raw' and 'imputed' assay slots are subsetted by remove_cluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:14ffa30a2db78e5ac8415b2579a0bb0882d644822903eef8446396307a5c0d69
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolomics-sample-subset-removal-from-multiassayexperiment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify outlier samples via combined hierarchical clustering heatmap annotated with k-means clusters (outlier_heatmap) and remove a flagged cluster from the raw and imputed assay slots of a MetaboDiff MultiAssayExperiment object using remove_cluster, prior to normalization.

## When to use

Use when a metabolomics MultiAssayExperiment object (post kNN imputation) shows a distinct outlier cluster in sample-sample clustering that should be excluded before normalization.

## When NOT to use

- The metabolite measurement table has not yet been merged into a MultiAssayExperiment object and imputed (knn_impute with 0.4 cutoff) — outlier_heatmap operates on the imputed measurements
- No distinct outlier cluster is apparent and removal would discard biologically meaningful samples without visual clustering evidence
- The goal is group-level differential testing (T-Test/ANOVA with Benjamini-Hochberg correction) rather than pre-normalization sample quality control

## Inputs

- MultiAssayExperiment object with 'raw' and 'imputed' slots from the MetaboDiff package (e.g. met_example after knn_impute with a 0.4 cutoff)
- Phenotype grouping factor (e.g. 'tumor_normal') and label_colors vector (e.g. c('darkseagreen','dodgerblue'))
- Number of k-means clusters k (e.g. k=2)

## Outputs

- Outlier heatmap PNG (e.g. outlier_heatmap.png) showing hierarchical clustering annotated with k-means cluster assignments
- Filtered MultiAssayExperiment object with the outlier cluster's samples removed from 'raw' and 'imputed' slots
- Object summary confirming the number of removed samples (e.g. 5)

## How to apply

Run outlier_heatmap on the imputed metabolite measurements of the MultiAssayExperiment object with group_factor set to the phenotype (e.g. 'tumor_normal'), label_colors matching the groups, and k=2 for the k-means clustering annotation. Inspect the heatmap to confirm which k-means cluster contains samples with on average lower relative metabolite measurements — in the MetaboDiff tutorial, cluster 1 contained 5 such samples. Then apply remove_cluster(met_example, cluster=1) to drop those samples from the 'raw' and 'imputed' slots of the MultiAssayExperiment object. Verify the returned object's summary confirms the expected number of samples was removed before proceeding to variance stabilizing normalization (vsn).

## Related tools

- **MetaboDiff** (R package providing outlier_heatmap and remove_cluster for outlier identification and cluster removal from the MultiAssayExperiment object) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment (R >= 4.0.2) in which outlier_heatmap and remove_cluster are executed)
- **MultiAssayExperiment** (Data container whose 'raw' and 'imputed' assay slots are subsetted by remove_cluster)

## Examples

```
met_example <- remove_cluster(met_example, cluster=1); outlier_heatmap(met_example$imputed, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue'), k=2)
```

## Evaluation signals

- The outlier heatmap reproduces a distinct outlier cluster visually separated in the hierarchical clustering (e.g. the 5-sample cluster 1 from the tutorial)
- The k-means annotation (k=2) assigns the visually flagged samples consistently to a single cluster
- Calling summary on the returned object confirms the expected number of samples was removed from the imputed data (5 in the tutorial example)
- Both 'raw' and 'imputed' slots show the reduced sample count, ensuring downstream normalization (vsn) operates on the filtered sample set
- Label colors correctly map the two groups ('darkseagreen'/'dodgerblue' for tumor/normal) in the heatmap annotation

## Limitations

- Requires R version 4.0.2 or higher for the MetaboDiff package
- Cluster removal depends on a subjective choice of k for the k-means annotation (tutorial uses k=2); a poor k may mis-assign outlier samples
- Removal is irreversible within the returned object; the original met_example must be reloaded if the wrong cluster is dropped

## Evidence
