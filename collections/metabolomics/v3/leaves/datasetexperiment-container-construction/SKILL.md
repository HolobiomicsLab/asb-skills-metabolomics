---
name: datasetexperiment-container-construction
description: Use when raw measured omics data (samples in rows, variables in columns)
  must be wrapped together with sample and variable metadata into a struct S4 DatasetExperiment
  object before being fed to struct model sequences.
when_to_use_negative:
- The data already exist as a DatasetExperiment (e.g. produced by iris_DatasetExperiment())
  — rebuilding the container risks breaking slot dimension consistency.
- The analysis target is a structToolbox model method (mean centring, PCA) rather
  than the data container itself — construct the container only as a prerequisite
  step.
- The data are not omics-style samples-by-variables matrices with accompanying metadata,
  so sample_meta/variable_meta slots cannot be populated consistently.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: struct
  role: Provides the S4 DatasetExperiment class templates and the formal iris dataset
    used to instantiate the container
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies model objects (mean_centre, PCA) exercised on the DatasetExperiment
    to confirm the container is valid and usable
- name: BiocManager
  role: Installation route for the struct package compatible with the current R version
- name: devtools
  role: Alternative installation of struct from GitHub via install_github
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:2f6122ad27e48c213eed189aa1f594c3ee94d355d59a0fb1858623020f15e091
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - BiocManager
  - Bioconductor
  - R
  - devtools
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# datasetexperiment-container-construction

## Summary

Constructs a struct S4 DatasetExperiment container that coherently binds a samples-by-variables measurement data.frame with sample_meta and variable_meta slots, so downstream struct/structToolbox models (e.g. mean_centre + PCA) can be trained and predicted via model_train/model_predict.

## When to use

Use when raw measured omics data (samples in rows, variables in columns) must be wrapped together with sample and variable metadata into a struct S4 DatasetExperiment object before being fed to struct model sequences.

## When NOT to use

- The data already exist as a DatasetExperiment (e.g. produced by iris_DatasetExperiment()) — rebuilding the container risks breaking slot dimension consistency.
- The analysis target is a structToolbox model method (mean centring, PCA) rather than the data container itself — construct the container only as a prerequisite step.
- The data are not omics-style samples-by-variables matrices with accompanying metadata, so sample_meta/variable_meta slots cannot be populated consistently.

## Inputs

- measurement data.frame with samples in rows and variables in columns (e.g. Fisher's iris 150 samples x 4 variables)
- sample_meta data.frame (one row per sample, e.g. 150 rows)
- variable_meta data.frame (one row per variable, e.g. 4 columns)

## Outputs

- valid S4 DatasetExperiment object with coordinated data/sample_meta/variable_meta slots
- optional serialized .rds file (e.g. iris_datasetexperiment.rds via saveRDS)

## How to apply

Load the struct package and either build the container manually or use the package's formal iris dataset. For manual construction, assign the 150x4 measurement matrix to the data slot, the 150-row sample metadata to sample_meta, and the 4-column variable metadata to variable_meta, then check the S4 object's validity so slot dimensions are consistent. The container's purpose is standardization: struct defines what inputs a method expects, what outputs it returns, and how methods combine into pipelines, so the DatasetExperiment is the expected object type for model_train and model_predict calls. Once populated and valid, the object can be serialized with saveRDS (e.g. iris_datasetexperiment.rds) or passed directly into a model sequence such as MS = mean_centre() + PCA(). Verify correctness by training a structToolbox model on it and retrieving outputs like scores and loadings via the output accessor.

## Related tools

- **struct** (Provides the S4 DatasetExperiment class templates and the formal iris dataset used to instantiate the container) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies model objects (mean_centre, PCA) exercised on the DatasetExperiment to confirm the container is valid and usable)
- **BiocManager** (Installation route for the struct package compatible with the current R version)
- **devtools** (Alternative installation of struct from GitHub via install_github)

## Examples

```
library(struct); DE = iris_DatasetExperiment()
```

## Evaluation signals

- Slot dimensions are consistent: data is 150 samples x 4 variables, sample_meta has 150 rows, variable_meta has 4 rows/entries for the iris container
- S4 validity check passes without slot-dimension errors
- The object is accepted by model_train and model_predict without type coercion errors
- Downstream outputs are retrievable: MS[2]$scores and MS[2]$loadings return non-empty matrices after a mean_centre + PCA sequence
- saveRDS/readRDS round-trip preserves the object's slot structure

## Limitations

- DatasetExperiment requires all three slots (data, sample_meta, variable_meta) to be dimensionally coordinated; mismatched slot dimensions invalidate the S4 object.
- struct is an R/Bioconductor package, so container construction is only available in R workflows (installed via BiocManager, conda/bioconda, or devtools from GitHub).
- The vignette context demonstrates the container with the exemplar iris dataset; other datasets must supply their own sample and variable metadata.

## Evidence

- [intro] formal iris dataset ships with struct: "A formal version of the iris dataset is included in the `struct` package."
- [intro] DatasetExperiment is the training input type: "A method used to train a model using a `DatasetExperiment` object."
- [readme] README quick-start container instantiation: "# Example dataset, in struct format
   DE = iris_DatasetExperiment()"
- [readme] model train/predict on the container: [evidence span withheld — anchor status: external_doc]
