---
name: bioconductor-example-dataset-access
description: Use when an analysis needs a small, citable LC-MS test dataset with matched
  feature quantification and sample metadata, or when xcms results (peak picking,
  grouping, retention-time correction) must be converted into one R object for downstream
  steps.
when_to_use_negative:
- The data already exist as a feature table or SummarizedExperiment, so re-accessing
  the raw example dataset would duplicate work
- The task is tuning xcms peak-picking or grouping parameters rather than structuring
  already-processed results
- The dataset is not available via Bioconductor/ExperimentHub (e.g., proprietary or
  non-R raw vendor files)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: environment in which the example dataset is loaded and the SummarizedExperiment
    is assembled and serialized
- name: faahKO
  role: Bioconductor example LC-MS dataset package supplying the raw samples for the
    workflow
- name: xcms
  role: performs peak picking, grouping, and retention-time correction and provides
    featureValues, featureDefinitions, and pData outputs
- name: SummarizedExperiment
  role: container class that holds the assay matrix with matching row and column annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:60476f89bf0af806fc4828fbac7ace6d616071be2578bf57916d5b5947b04d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - faahKO
  - xcms
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# bioconductor-example-dataset-access

## Summary

Retrieves a Bioconductor example metabolomics dataset (e.g., the faahKO LC-MS data used in Section 2.1.6 of the metaRbolomics review) and structures the xcms pre-processing outputs — featureValues assay, featureDefinitions, and pData — into a single coherent SummarizedExperiment saved as .rds.

## When to use

Use when an analysis needs a small, citable LC-MS test dataset with matched feature quantification and sample metadata, or when xcms results (peak picking, grouping, retention-time correction) must be converted into one R object for downstream steps.

## When NOT to use

- The data already exist as a feature table or SummarizedExperiment, so re-accessing the raw example dataset would duplicate work
- The task is tuning xcms peak-picking or grouping parameters rather than structuring already-processed results
- The dataset is not available via Bioconductor/ExperimentHub (e.g., proprietary or non-R raw vendor files)

## Inputs

- Bioconductor example dataset package (e.g., faahKO LC-MS raw samples in NetCDF/CDF format)
- XCMSnExp object with completed xcms pre-processing (peak picking, grouping, retention-time correction)

## Outputs

- SummarizedExperiment object with featureValues assay, featureDefinitions as rowData, and pData as colData
- serialized .rds file (summarized_experiment.rds)

## How to apply

Load the faahKO example LC-MS dataset from Bioconductor and run (or load) the xcms pre-processing results (peak picking, grouping, retention-time correction) on the raw samples. Extract the quantitative feature matrix via the featureValues output of the XCMSnExp object to serve as the assay data. Extract the featureDefinitions table as rowData (row annotations) and the pData sample metadata as colData (column annotations), assembling them into a SummarizedExperiment with matching row and column dimensions. Serialize the object to disk with saveRDS as summarized_experiment.rds; the rationale is that combining reusable data processing R scripts with the experimental data supports open, reproducible research.

## Related tools

- **R** (environment in which the example dataset is loaded and the SummarizedExperiment is assembled and serialized)
- **faahKO** (Bioconductor example LC-MS dataset package supplying the raw samples for the workflow)
- **xcms** (performs peak picking, grouping, and retention-time correction and provides featureValues, featureDefinitions, and pData outputs)
- **SummarizedExperiment** (container class that holds the assay matrix with matching row and column annotations)

## Examples

```
library(faahKO); data(faahko); se <- readRDS("summarized_experiment.rds")
```

## Evaluation signals

- The .rds file (summarized_experiment.rds) exists on disk after saveRDS
- The R script runs to completion without errors
- The SummarizedExperiment contains the assay matrix (featureValues), rowData (featureDefinitions), and colData (pData)
- Row and column dimensions are consistent: nrow(assay) matches nrow(rowData) and ncol(assay) matches nrow(colData)
- Re-loading the .rds with readRDS restores an object of class SummarizedExperiment with the expected faahKO sample count

## Limitations

- Requires xcms pre-processing (peak picking, grouping, retention-time correction) to have been run or loadable on the raw samples first; missing peaks can yield NA entries in featureValues
- Depends on the Bioconductor example dataset (faahKO) remaining available at the installed Bioconductor version
- The review is an overview of R packages, so it prescribes structure rather than instrument- or dataset-specific pre-processing parameters

## Evidence

- [methods] A SummarizedExperiment can be generated from xcms pre-processing results by adding the featureValues output as the quantitative assay: [evidence span withheld — anchor status: not_found]
- [methods] faahKO load step: [evidence span withheld — anchor status: not_found]
- [abstract] open reproducible research: "Combining reusable data processing R scripts with the experimental data thus allows for open, reproducible research."
- [abstract] R popularity for metabolomics: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
