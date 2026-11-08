---
name: r-object-serialization-rds
description: Use when raw LC-MS samples have been pre-processed in xcms (peak picking,
  grouping, retention-time correction) and the quantitative feature matrix, feature
  annotations, and sample metadata must be combined into one coherent R object for
  downstream analysis or archiving.
when_to_use_negative:
- Input is already a processed feature table with annotations (no xcms XCMSnExp object
  exists to extract featureValues/featureDefinitions/pData from)
- The analysis requires raw spectrometry data access (chromatograms, spectra) that
  a serialized SummarizedExperiment does not retain
- Downstream tools expect an XCMSnExp object rather than a SummarizedExperiment container
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Programming/statistics environment in which the xcms outputs are assembled
    into the SummarizedExperiment and serialized with saveRDS
- name: xcms
  role: Performs the LC-MS pre-processing (peak picking, grouping, retention-time
    correction) and provides featureValues, featureDefinitions, and pData as inputs
    to the container
- name: SummarizedExperiment
  role: Bioconductor container that holds the assay matrix together with row (feature)
    and column (sample) annotations as a single coherent object
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:07f6ef86deb2a3878b36150a3c41a2481ad22c5be34eebf2f110cc193e691804
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - faahKO
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# r-object-serialization-rds

## Summary

Structures xcms pre-processing results (featureValues assay, featureDefinitions rowData, pData colData) into a single SummarizedExperiment object and serializes it to disk as an .rds file, enabling open, reproducible metabolomics data processing in R.

## When to use

Use when raw LC-MS samples have been pre-processed in xcms (peak picking, grouping, retention-time correction) and the quantitative feature matrix, feature annotations, and sample metadata must be combined into one coherent R object for downstream analysis or archiving.

## When NOT to use

- Input is already a processed feature table with annotations (no xcms XCMSnExp object exists to extract featureValues/featureDefinitions/pData from)
- The analysis requires raw spectrometry data access (chromatograms, spectra) that a serialized SummarizedExperiment does not retain
- Downstream tools expect an XCMSnExp object rather than a SummarizedExperiment container

## Inputs

- XCMSnExp object from xcms pre-processing (peak picking, grouping, retention-time correction) of raw LC-MS samples
- faahKO example LC-MS dataset from Bioconductor (or user raw mzML/NetCDF LC-MS runs)
- featureValues quantitative feature matrix
- featureDefinitions feature table
- pData sample metadata table

## Outputs

- SummarizedExperiment object containing the featureValues assay, featureDefinitions rowData, and pData colData with consistent dimensions
- Serialized .rds file (e.g., summarized_experiment.rds) written via saveRDS

## How to apply

Run (or load) the xcms pre-processing steps — peak picking, grouping, and retention-time correction — on the raw samples (e.g., the Bioconductor faahKO example dataset) to obtain an XCMSnExp object. Extract featureValues as the quantitative assay matrix, featureDefinitions as rowData, and pData as colData, then assemble these into a SummarizedExperiment whose row and column dimensions match (features × samples). Serialize the object with saveRDS to an .rds file such as summarized_experiment.rds so that the processed data and its metadata travel together. This follows the metaRbolomics rationale that combining reusable R data-processing scripts with experimental data enables open, reproducible research.

## Related tools

- **R** (Programming/statistics environment in which the xcms outputs are assembled into the SummarizedExperiment and serialized with saveRDS)
- **xcms** (Performs the LC-MS pre-processing (peak picking, grouping, retention-time correction) and provides featureValues, featureDefinitions, and pData as inputs to the container)
- **SummarizedExperiment** (Bioconductor container that holds the assay matrix together with row (feature) and column (sample) annotations as a single coherent object)

## Examples

```
Rscript -e 'se <- SummarizedExperiment(assays=featureValues(xset), rowData=featureDefinitions(xset), colData=pData(xset)); saveRDS(se, "summarized_experiment.rds")'
```

## Evaluation signals

- The .rds file (e.g., summarized_experiment.rds) exists on disk after the R script runs to completion
- The SummarizedExperiment contains a non-empty assay matrix matching featureValues dimensions
- rowData corresponds to the featureDefinitions table and colData to the pData sample annotations
- Row (feature) and column (sample) dimensions of assay, rowData, and colData are mutually consistent
- Reading the .rds back with readRDS returns an object with identical assay, rowData, and colData content

## Limitations

- The container reflects the state after xcms pre-processing only; re-running peak picking, grouping, or retention-time correction requires regenerating the object from the raw samples
- Serialization assumes matching feature/sample dimensions between featureValues, featureDefinitions, and pData; mismatches will break assembly
- The review catalogs hundreds of metabolomics R packages across CRAN, Bioconductor, and GitHub, so tool availability and versions may vary across those repositories

## Evidence

- [abstract] R environment for reproducible metabolomics processing: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] Reproducible data + script combination: "Combining reusable data processing R scripts with the experimental data thus allows for open, reproducible research."
- [other] SummarizedExperiment assembly from xcms outputs: [evidence span withheld — anchor status: not_found]
