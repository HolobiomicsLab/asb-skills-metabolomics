---
name: metabolite-missingness-visualization
description: Use when a metabolomics measurement table contains missing values and
  you need to inspect missingness patterns per metabolite/sample before imputation,
  and to decide which metabolites to exclude (missingness > 40%) prior to differential
  analysis.
when_to_use_negative:
- The dataset has already undergone imputation or contains no missing measurements
  — na_heatmap and knn_impute would be redundant.
- You need imputation of metabolites with very high missingness (> 40%) rather than
  their exclusion; the MetaboDiff workflow explicitly drops these instead of imputing
  them.
- Your data are not a table of metabolite measurements packaged as a MultiAssayExperiment,
  which is the input format MetaboDiff expects.
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MetaboDiff
  role: Provides the na_heatmap function for missingness visualization and the knn_impute
    function with the 40% (0.4) cutoff
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container object type (created by create_mae) whose assay slots hold the metabolite
    measurement tables processed by na_heatmap and knn_impute
- name: R
  role: Runtime environment required at version 4.0.2 or higher to install and run
    MetaboDiff
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:0319a55e06dee3434c3dead4729211bb144956a2c7d955d76a1ed1b87c411dd8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-missingness-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize the fraction of missing measurements per metabolite and per sample as a binary heatmap with barplots (MetaboDiff's na_heatmap), then apply k-nearest-neighbor imputation restricted to metabolites missing in at most 40% of samples (knn_impute, cutoff = 0.4).

## When to use

Use when a metabolomics measurement table contains missing values and you need to inspect missingness patterns per metabolite/sample before imputation, and to decide which metabolites to exclude (missingness > 40%) prior to differential analysis.

## When NOT to use

- The dataset has already undergone imputation or contains no missing measurements — na_heatmap and knn_impute would be redundant.
- You need imputation of metabolites with very high missingness (> 40%) rather than their exclusion; the MetaboDiff workflow explicitly drops these instead of imputing them.
- Your data are not a table of metabolite measurements packaged as a MultiAssayExperiment, which is the input format MetaboDiff expects.

## Inputs

- MetaboDiff MultiAssayExperiment object of metabolite measurements with a missing values slot (e.g. case1_cells, case1_mice, case1_human from Priolo et al. 2014)
- group_factor column (e.g. 'group') defining sample grouping for the heatmap barplots

## Outputs

- Binary heatmap with barplots visualizing the fraction of missing measurements per metabolite and per sample
- Imputed MultiAssayExperiment slot containing only metabolites missing in <= 40% of samples (e.g. 69 metabolites excluded in the Priolo example data)

## How to apply

Load the MetaboDiff package in R (>= 4.0.2) and operate on the bundled MultiAssayExperiment case-study objects (case1_cells, case1_mice, case1_human from Priolo et al. 2014). Call na_heatmap with group_factor='group' to render a binary heatmap of missing measurements per metabolite and per sample, accompanied by barplots of the missingness fraction. Then apply knn_impute with cutoff=0.4, so that k-nearest-neighbor imputation is performed only for metabolites with raw measurements in more than 60% of samples; metabolites missing in more than 40% of samples are excluded. This ordering (inspect, then impute with the 40% cutoff) is the MetaboDiff-recommended strategy for handling missing values in quantitative metabolomic datasets.

## Related tools

- **MetaboDiff** (Provides the na_heatmap function for missingness visualization and the knn_impute function with the 40% (0.4) cutoff) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container object type (created by create_mae) whose assay slots hold the metabolite measurement tables processed by na_heatmap and knn_impute)
- **R** (Runtime environment required at version 4.0.2 or higher to install and run MetaboDiff)

## Examples

```
na_heatmap(case1_mae, group_factor = "group"); case1_mae <- knn_impute(case1_mae, cutoff = 0.4)
```

## Evaluation signals

- na_heatmap output is a binary heatmap of missing measurements with barplots, stratified by group_factor='group'.
- knn_impute runs with cutoff=0.4; the imputed slot must contain no metabolite missing in more than 40% of samples.
- In the Priolo 2014 example data, the imputed slot excludes exactly 69 metabolites, matching the tutorial's reported count.
- Imputation is applied only to metabolites with raw measurements present in more than 60% of samples.

## Limitations

- Metabolites missing in more than 40% of samples are excluded rather than imputed, so highly sparse metabolites cannot be recovered by this workflow.
- The package requires R >= 4.0.2 and input organized as a MultiAssayExperiment object.
- The threshold (40%) is a recommendation from the MetaboDiff authors and may need adjustment for datasets with different missingness profiles.

## Evidence
