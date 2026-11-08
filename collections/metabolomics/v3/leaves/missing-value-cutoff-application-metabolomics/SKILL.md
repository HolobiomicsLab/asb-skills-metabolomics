---
name: missing-value-cutoff-application-metabolomics
description: Use when a metabolite-by-sample measurement table contains missing values
  and metabolites must be filtered by a missingness cutoff before imputation and downstream
  differential analysis.
when_to_use_negative:
- The dataset has already been imputed or contains no missing metabolite measurements
  — applying the 40% cutoff again would drop no rows but adds no value
- The input is a single-group or non-quantitative metabolite table where a group_factor
  for the na_heatmap missingness visualization cannot be supplied
- The analysis goal is correlation-network or differential testing on raw (non-imputed)
  data and missingness filtering was already performed upstream
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDiff
  role: Provides the na_heatmap function to visualize missingness and the knn_impute
    function that performs k-nearest-neighbor imputation with the 40% (0.4) cutoff
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment; MetaboDiff requires R version 4.0.2 or higher
- name: MultiAssayExperiment
  role: Container object format (created via create_mae) whose imputed slot receives
    the kNN-imputed, cutoff-filtered metabolite measurements
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:9f7045829562bbdd5d861047e8e7129a86c50206b1b992f88b95fc7babde5c8f
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# missing-value-cutoff-application-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitative metabolomics datasets require explicit handling of missing measurements before differential analysis; MetaboDiff visualizes missingness with a binary heatmap (na_heatmap) and applies k-nearest-neighbor imputation (knn_impute) only to metabolites measured in more than 60% of samples.

## When to use

Use when a metabolite-by-sample measurement table contains missing values and metabolites must be filtered by a missingness cutoff before imputation and downstream differential analysis.

## When NOT to use

- The dataset has already been imputed or contains no missing metabolite measurements — applying the 40% cutoff again would drop no rows but adds no value
- The input is a single-group or non-quantitative metabolite table where a group_factor for the na_heatmap missingness visualization cannot be supplied
- The analysis goal is correlation-network or differential testing on raw (non-imputed) data and missingness filtering was already performed upstream

## Inputs

- Metabolite-by-sample quantitative measurement table stored in a MultiAssayExperiment object (e.g. the Priolo 2014 case1_cells, case1_mice, or case1_human datasets)
- Sample grouping factor (e.g. 'group' column) used to stratify the missingness heatmap

## Outputs

- Binary missingness heatmap with barplots of missing fraction per metabolite and per sample
- MultiAssayExperiment object with an imputed slot containing kNN-imputed values for metabolites with <= 40% missingness and removal of metabolites exceeding the cutoff

## How to apply

First load the MetaboDiff package in R (>= 4.0.2) and inspect missingness per metabolite and per sample by calling na_heatmap on the MultiAssayExperiment object with group_factor set to the grouping column (e.g. 'group'), producing a binary heatmap with accompanying barplots. Then apply knn_impute with cutoff=0.4, following the tutorial's recommended cutoff of 40%, so that k-nearest-neighbor imputation is performed only on metabolites with raw measurements in more than 60% of samples; metabolites exceeding the threshold are excluded from imputation entirely. Verify the filtering worked by confirming the imputed slot drops the over-threshold metabolites — in the Priolo et al. 2014 case-study data (case1_cells, case1_mice, case1_human) this corresponds to the tutorial's reported 69 excluded metabolites.

## Related tools

- **MetaboDiff** (Provides the na_heatmap function to visualize missingness and the knn_impute function that performs k-nearest-neighbor imputation with the 40% (0.4) cutoff) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment; MetaboDiff requires R version 4.0.2 or higher)
- **MultiAssayExperiment** (Container object format (created via create_mae) whose imputed slot receives the kNN-imputed, cutoff-filtered metabolite measurements)

## Examples

```
case1_cells_imputed <- knn_impute(case1_cells, group_factor = "group", cutoff = 0.4)
```

## Evaluation signals

- The imputed slot contains no metabolite with missing measurements in more than 40% of samples (cutoff=0.4 respected)
- For the bundled Priolo 2014 case-study data, the number of excluded metabolites equals the tutorial's reported 69
- na_heatmap output renders as a binary heatmap with barplots showing the fraction of missing measurements per metabolite and per sample
- Metabolites retained for imputation have raw measurements in more than 60% of samples
- The R session runs MetaboDiff under R >= 4.0.2 as required by the package

## Limitations

- Metabolites missing in more than 40% of samples are permanently excluded before imputation, so information loss is accepted rather than recovered
- kNN imputation is only appropriate for quantitative metabolomic measurement tables; its behavior on other data types is not covered by the tutorial
- No changelog was found in the repository, so version-specific behavior changes of knn_impute cannot be traced

## Evidence
