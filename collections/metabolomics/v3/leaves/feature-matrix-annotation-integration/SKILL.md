---
name: feature-matrix-annotation-integration
description: Use when xcms pre-processing (peak picking, grouping, retention-time
  correction) has produced an XCMSnExp object on LC-MS samples and you need the feature
  matrix plus its feature and sample metadata bundled into one R object.
when_to_use_negative:
- Input is already a structured feature table with its annotations, not raw xcms output
- The pre-processing (peak picking, grouping, RT correction) has not yet been run
  and no XCMSnExp results exist
- Metadata and feature matrix dimensions cannot be reconciled (feature rows / sample
  columns mismatch), since assembly requires consistent dimensions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Programming/statistics environment in which the data structuring and SummarizedExperiment
    assembly are performed
- name: xcms
  role: Performs the pre-processing (peak picking, grouping, retention-time correction)
    whose featureValues, featureDefinitions and pData outputs are restructured
- name: SummarizedExperiment
  role: Provides the container class holding the quantitative assay, rowData and colData
    annotations
- name: faahKO
  role: Bioconductor example LC-MS dataset used as the input raw samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:03b43e434d1aefe06aa1223b4094e22950ec237a395a9e3b4ae4f4d9db7d48e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - faahKO
  - xcms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# feature-matrix-annotation-integration

## Summary

Assemble xcms pre-processing results (featureValues, featureDefinitions, pData) into a single SummarizedExperiment object so quantitative feature data and its metadata travel together as one coherent R object for downstream metabolomics analysis.

## When to use

Use when xcms pre-processing (peak picking, grouping, retention-time correction) has produced an XCMSnExp object on LC-MS samples and you need the feature matrix plus its feature and sample metadata bundled into one R object.

## When NOT to use

- Input is already a structured feature table with its annotations, not raw xcms output
- The pre-processing (peak picking, grouping, RT correction) has not yet been run and no XCMSnExp results exist
- Metadata and feature matrix dimensions cannot be reconciled (feature rows / sample columns mismatch), since assembly requires consistent dimensions

## Inputs

- XCMSnExp object from xcms pre-processing of raw LC-MS samples (e.g., faahKO NetCDF files)
- featureValues matrix from the XCMSnExp object
- featureDefinitions table of feature metadata
- pData sample metadata table

## Outputs

- SummarizedExperiment object with assay (featureValues), rowData (featureDefinitions), colData (pData)
- .rds file (e.g., summarized_experiment.rds) serialized with saveRDS

## How to apply

Run (or load) the xcms pre-processing results — peak picking, grouping and retention-time correction — on raw LC-MS samples such as the faahKO example dataset from Bioconductor. Extract the featureValues output of the XCMSnExp object as the quantitative assay matrix. Take the featureDefinitions table as rowData (feature annotations) and the pData sample metadata as colData (column annotations). Assemble these three components into a SummarizedExperiment with matching row and column dimensions, checking that the number of features in featureValues matches featureDefinitions and the number of columns matches pData rows. Serialize the object to disk with saveRDS (e.g., summarized_experiment.rds) so the structured result can be reused or shared for reproducible research.

## Related tools

- **R** (Programming/statistics environment in which the data structuring and SummarizedExperiment assembly are performed)
- **xcms** (Performs the pre-processing (peak picking, grouping, retention-time correction) whose featureValues, featureDefinitions and pData outputs are restructured)
- **SummarizedExperiment** (Provides the container class holding the quantitative assay, rowData and colData annotations)
- **faahKO** (Bioconductor example LC-MS dataset used as the input raw samples)

## Examples

```
se <- SummarizedExperiment(assays=featureValues(xset), rowData=featureDefinitions(xset), colData=pData(xset)); saveRDS(se, "summarized_experiment.rds")
```

## Evaluation signals

- The .rds output file exists on disk after saveRDS
- The R script runs to completion without error
- The SummarizedExperiment contains the assay matrix (featureValues)
- rowData holds the featureDefinitions table
- colData holds the pData sample annotations with consistent dimensions matching the assay

## Limitations

- Assembly requires featureDefinitions rows and pData columns to align exactly with the featureValues dimensions; mismatched pre-processing results will break the object
- The skill applies to post-pre-processing xcms output only; it does not cover raw data conversion or the pre-processing itself

## Evidence

- [methods] structuring step from Section 2.1.6 workflow: [evidence span withheld — anchor status: not_found]
- [abstract] reproducibility rationale: "Combining reusable data processing R scripts with the experimental data thus allows for open, reproducible research."
- [abstract] R as the metabolomics processing environment: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
