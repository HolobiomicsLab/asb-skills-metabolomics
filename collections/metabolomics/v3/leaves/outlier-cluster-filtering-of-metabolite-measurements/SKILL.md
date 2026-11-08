---
name: outlier-cluster-filtering-of-metabolite-measurements
description: Use when sample-level outliers distort the metabolite measurement table
  — e.g. the outlier heatmap shows samples grouping into a distinct k-means cluster
  that should be removed prior to normalization or differential testing.
when_to_use_negative:
- The measurement table is a raw vendor file that still needs create_mae assembly
  or missing-value imputation — imputation (knn_impute, cutoff 0.4) must precede cluster
  assessment
- The goal is differential testing or network analysis on an already quality-filtered
  sample set
- No sample-level outlier structure is visible — removing an arbitrary cluster would
  discard valid cohort samples
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: Provides remove_cluster and the outlier heatmap used to detect and drop outlier
    sample clusters from the MultiAssayExperiment
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container object type that remove_cluster operates on, merging the metabolite
    measurements with sample metadata
- name: R
  role: Runtime environment; MetaboDiff requires R version 4.0.2 or higher
- name: Metabolon®
  role: Commercial metabolomics platform whose measurement tables form the exemplar
    input data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:648d1535e49f78e9c1107484cc673242c52750df4beb9764adfd98f366894236
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
  - Metabolon®
  - MultiAssayExperiment
  - R
  - vsn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# outlier-cluster-filtering-of-metabolite-measurements

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters outlier sample clusters from a metabolomics MultiAssayExperiment by k-means clustering of the measurement data and removing an identified cluster (e.g. cluster 1) via MetaboDiff's remove_cluster function before downstream differential analysis.

## When to use

Use when sample-level outliers distort the metabolite measurement table — e.g. the outlier heatmap shows samples grouping into a distinct k-means cluster that should be removed prior to normalization or differential testing.

## When NOT to use

- The measurement table is a raw vendor file that still needs create_mae assembly or missing-value imputation — imputation (knn_impute, cutoff 0.4) must precede cluster assessment
- The goal is differential testing or network analysis on an already quality-filtered sample set
- No sample-level outlier structure is visible — removing an arbitrary cluster would discard valid cohort samples

## Inputs

- MultiAssayExperiment object produced by create_mae and knn_impute (missing-value cutoff 0.4)
- Table of metabolite measurements (e.g. Metabolon® platform output)
- Chosen number of k-means clusters (k, e.g. k=2 in the MetaboDiff example data)

## Outputs

- MultiAssayExperiment object with the outlier cluster (e.g. cluster 1) removed
- Outlier heatmap figure showing k-means clustering of samples before removal

## How to apply

After knn_impute with the recommended 40% (0.4) missing-value cutoff, generate the MetaboDiff outlier heatmap to inspect k-means clustering of samples (the exemplar data sets k=2). Identify the cluster of outlier samples and call remove_cluster to drop it from the MultiAssayExperiment (in the tutorial, cluster 1 is removed). The cleaned object — still structured as a MultiAssayExperiment over Metabolon®-style metabolite measurements — then feeds the variance stabilizing normalization step. Rationale: samples clustering apart from the study cohort can dominate the measurement spectrum and bias downstream T-Test/ANOVA comparisons.

## Related tools

- **MetaboDiff** (Provides remove_cluster and the outlier heatmap used to detect and drop outlier sample clusters from the MultiAssayExperiment) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container object type that remove_cluster operates on, merging the metabolite measurements with sample metadata)
- **R** (Runtime environment; MetaboDiff requires R version 4.0.2 or higher)
- **Metabolon®** (Commercial metabolomics platform whose measurement tables form the exemplar input data)

## Examples

```
metabo_mae <- remove_cluster(metabo_mae, cluster = 1)
```

## Evaluation signals

- The outlier heatmap shows the removed cluster's samples no longer present in the k-means plot of the retained cohort
- The retained object is still a valid MultiAssayExperiment with sample metadata intact after remove_cluster
- Imputation was performed before clustering (missing values handled with the recommended 0.4 cutoff), matching the documented workflow order
- Downstream quality_plot on the normalized object shows no single outlier sample dominating the measurement spectrum

## Limitations

- The number of clusters (k) and which cluster to remove are manual decisions guided by the outlier heatmap (k=2 in the exemplar data), so misjudgment can drop genuine cohort samples
- The tutorial workflow applies this filtering to the MetaboDiff example dataset; behavior on other platforms or sample sizes is not characterized in the article
- No changelog found in the repository, so version-specific behavior of remove_cluster is undocumented

## Evidence
