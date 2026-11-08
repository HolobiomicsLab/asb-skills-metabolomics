---
name: s4-object-construction-in-r
description: Use when measured omics data (samples in rows, variables in columns)
  plus its sample metadata and variable metadata must be assembled into a single validated
  S4 object in R/struct for downstream model_train/model_predict workflows.
when_to_use_negative:
- When the data are already held in a struct-compatible object returned by a constructor
  such as iris_DatasetExperiment() — repopulating slots manually would be redundant
- When the analysis target is model chaining (e.g. mean centring followed by PCA)
  rather than container construction — use the model_train/model_predict workflow
  instead
- When working outside R/Bioconductor — struct templates are S4/R-based and cannot
  be applied directly
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: struct
  role: Provides the S4 class-based templates and the DatasetExperiment container
    (data, sample_meta, variable_meta slots) being constructed
  repo: https://github.com/computational-metabolomics
- name: R
  role: Runtime language in which the S4 object is instantiated, validity-checked,
    and serialized with saveRDS
- name: BiocManager
  role: Used to install the struct package from Bioconductor before constructing objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:341baa8cc43c91ba7d55f849c767cff067e0251675b7290f5ce43cbf28d2b3fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor
  - R
  - struct
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# s4-object-construction-in-r

## Summary

Constructs validated S4 objects in R using the struct framework's class-based templates, specifically populating the DatasetExperiment container with coordinated data, sample_meta, and variable_meta slots for omics measurements such as Fisher's 150x4 iris dataset.

## When to use

Use when measured omics data (samples in rows, variables in columns) plus its sample metadata and variable metadata must be assembled into a single validated S4 object in R/struct for downstream model_train/model_predict workflows.

## When NOT to use

- When the data are already held in a struct-compatible object returned by a constructor such as iris_DatasetExperiment() — repopulating slots manually would be redundant
- When the analysis target is model chaining (e.g. mean centring followed by PCA) rather than container construction — use the model_train/model_predict workflow instead
- When working outside R/Bioconductor — struct templates are S4/R-based and cannot be applied directly

## Inputs

- measurement data.frame with samples in rows and variables in columns (e.g. Fisher's iris 150x4 matrix)
- sample metadata data.frame (150 rows)
- variable metadata data.frame (4 rows/columns)

## Outputs

- validated S4 DatasetExperiment object
- serialized .rds file (iris_datasetexperiment.rds via saveRDS)

## How to apply

Install struct via BiocManager or devtools::install_github('computational-metabolomics/struct'), then load it with library(struct). Instantiate the formal DatasetExperiment-ready dataset shipped with the package via iris_DatasetExperiment(), or populate the three DatasetExperiment slots manually: assign the 150x4 measurement data.frame to the data slot, the 150-row sample metadata to sample_meta, and the 4-column variable metadata to variable_meta. Run the S4 validity check so slot dimensions are consistent (150 samples x 4 variables) before proceeding. Serialize the completed object with saveRDS to produce iris_datasetexperiment.rds, or hand it directly to model_train/model_predict as shown in the README quick-start pipeline (mean_centre() + PCA()).

## Related tools

- **struct** (Provides the S4 class-based templates and the DatasetExperiment container (data, sample_meta, variable_meta slots) being constructed) — https://github.com/computational-metabolomics
- **R** (Runtime language in which the S4 object is instantiated, validity-checked, and serialized with saveRDS)
- **BiocManager** (Used to install the struct package from Bioconductor before constructing objects)

## Examples

```
library(struct); DE = iris_DatasetExperiment(); saveRDS(DE, 'iris_datasetexperiment.rds')
```

## Evaluation signals

- validObject() on the DatasetExperiment passes, confirming slot dimensions are consistent (150 samples x 4 variables for iris)
- data slot is a 150x4 data.frame, sample_meta has 150 rows, variable_meta has 4 rows
- saveRDS/readRDS round-trip reproduces an identical object
- The object can be passed to model_train(MS, DE) and model_predict(MS, DE) without errors, and outputs such as scores and loadings are retrievable via MS[2]$scores

## Limitations

- Slot dimensions must be consistent or the S4 validity check fails; mismatched sample_meta/data row counts are a failure mode
- struct defines templates rather than a single fixed container, so users must know which template (e.g. DatasetExperiment) fits their data layout
- The companion structToolbox package is needed for many downstream methods (mean_centre, PCA); struct alone provides only the object templates

## Evidence

- [intro] object-oriented templates define container construction: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
- [intro] formal iris dataset shipped with struct: "A formal version of the iris dataset is included in the `struct` package."
- [abstract] class-based templates rationale: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows"
- [readme] README quick-start instantiation of DatasetExperiment: "# Example dataset, in struct format
   DE = iris_DatasetExperiment()"
- [intro] training on DatasetExperiment objects: "A method used to train a model using a `DatasetExperiment` object."
