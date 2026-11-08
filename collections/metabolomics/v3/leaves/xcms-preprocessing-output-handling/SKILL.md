---
name: xcms-preprocessing-output-handling
description: Use when xcms peak picking, grouping, and retention-time correction have
  been run on raw LC-MS samples (e.g. the faahKO dataset) and the feature matrix must
  be combined with feature and sample metadata into one coherent R object.
when_to_use_negative:
- Input is already a plain feature table without xcms featureDefinitions/pData outputs
  — no annotations exist to fill rowData/colData.
- The analysis goal is raw spectra exploration or peak visualization rather than assembling
  a quantitative feature assay.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Programming/statistics environment in which the SummarizedExperiment is assembled
    and serialized
- name: xcms
  role: Produces the XCMSnExp pre-processing outputs (featureValues, featureDefinitions)
    used as assay and rowData
- name: SummarizedExperiment
  role: Bioconductor container class holding the assay matrix plus row and column
    annotations
- name: faahKO
  role: Bioconductor example LC-MS dataset used as the demonstration input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:0b7b87d8328553ebf61b9e729e09a612cd26c6d58fe1311b824df4daeceb6c4a
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# xcms-preprocessing-output-handling

## Summary

Structures xcms LC-MS pre-processing results (featureValues, featureDefinitions, pData from an XCMSnExp object) into a single Bioconductor SummarizedExperiment with a quantitative assay plus feature- and sample-level annotations, serialized as an .rds file for downstream reproducible use.

## When to use

Use when xcms peak picking, grouping, and retention-time correction have been run on raw LC-MS samples (e.g. the faahKO dataset) and the feature matrix must be combined with feature and sample metadata into one coherent R object.

## When NOT to use

- Input is already a plain feature table without xcms featureDefinitions/pData outputs — no annotations exist to fill rowData/colData.
- The analysis goal is raw spectra exploration or peak visualization rather than assembling a quantitative feature assay.

## Inputs

- XCMSnExp object (or xcms pre-processing results) containing peak picking, grouping, and retention-time correction outputs on raw LC-MS samples
- faahKO example LC-MS dataset from Bioconductor (or user LC-MS raw data processed identically)
- R script to build and serialize the container

## Outputs

- SummarizedExperiment object with featureValues as the assay, featureDefinitions as rowData, and pData as colData with consistent dimensions
- Serialized .rds file (e.g. summarized_experiment.rds) on disk

## How to apply

Load the faahKO example LC-MS dataset from Bioconductor and run (or load) the xcms pre-processing results — peak picking, grouping, and retention-time correction — on the raw samples. Extract the quantitative feature matrix from the XCMSnExp object via its featureValues output to serve as the assay data. Extract featureDefinitions as rowData (row annotations) and pData sample metadata as colData (column annotations). Assemble these three components into a SummarizedExperiment with matching row and column dimensions, then serialize it with saveRDS (e.g. summarized_experiment.rds). The rationale, grounded in the metaRbolomics review, is that Bioconductor's S4 container ecosystem lets a single R object carry data plus annotations, supporting open, reproducible research when combined with reusable R scripts.

## Related tools

- **R** (Programming/statistics environment in which the SummarizedExperiment is assembled and serialized)
- **xcms** (Produces the XCMSnExp pre-processing outputs (featureValues, featureDefinitions) used as assay and rowData)
- **SummarizedExperiment** (Bioconductor container class holding the assay matrix plus row and column annotations)
- **faahKO** (Bioconductor example LC-MS dataset used as the demonstration input)

## Examples

```
se <- SummarizedExperiment(assays = featureValues(xset), rowData = featureDefinitions(xset), colData = pData(xset)); saveRDS(se, "summarized_experiment.rds")
```

## Evaluation signals

- The .rds file (e.g. summarized_experiment.rds) exists on disk after saveRDS.
- The R script runs to completion without errors.
- The object contains the assay matrix (featureValues), rowData feature definitions, and colData sample annotations.
- Row and column dimensions of the SummarizedExperiment match the number of xcms features and samples respectively.

## Limitations

- Requires completed xcms pre-processing (peak picking, grouping, retention-time correction); it cannot be applied to raw spectra alone.
- Tied to Bioconductor S4 container conventions; datasets without pData-style sample metadata will leave colData incomplete.

## Evidence

- [abstract] R as metabolomics environment: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] Bioconductor repository context: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [abstract] reproducible research rationale: "Combining reusable data processing R scripts with the experimental data thus allows for open, reproducible research."
