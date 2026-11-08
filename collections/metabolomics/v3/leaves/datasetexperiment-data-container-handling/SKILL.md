---
name: datasetexperiment-data-container-handling
description: Use when implementing or running struct/structToolbox models in R and
  the input data must be coerced into the DatasetExperiment format that model_train
  and model_predict expect.
when_to_use_negative:
- The analysis is not being done in R with the struct/structToolbox framework and
  does not need S4 template objects — plain data frames or matrices are simpler
- The data is single-table metadata or results that no struct model will consume,
  so the DatasetExperiment wrapper adds overhead without benefit
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: struct
  role: Provides the DatasetExperiment container class and the model_train/model_predict
    methods that operate on it
  repo: github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies ready-made models (mean_centre, PCA) that consume DatasetExperiment
    objects in pipelines
  repo: github.com/computational-metabolomics/struct
- name: R
  role: Language in which DatasetExperiment objects are created, trained, and predicted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:f2e67a9ebf2fd686fe8d144dfb5295b71e23f84eff0bf55949d64c40b5542ec7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# DatasetExperiment data container handling

## Summary

Wraps tabular 'omics data in struct's DatasetExperiment S4 container so that model_train/model_predict methods can consume it uniformly; exemplified by converting Fisher's iris dataset via iris_DatasetExperiment() and passing it through mean-centring and PCA models.

## When to use

Use when implementing or running struct/structToolbox models in R and the input data must be coerced into the DatasetExperiment format that model_train and model_predict expect.

## When NOT to use

- The analysis is not being done in R with the struct/structToolbox framework and does not need S4 template objects — plain data frames or matrices are simpler
- The data is single-table metadata or results that no struct model will consume, so the DatasetExperiment wrapper adds overhead without benefit

## Inputs

- R data as a struct DatasetExperiment S4 object (e.g. iris_DatasetExperiment(), a formal version of Fisher's iris dataset bundled with struct)
- A struct model or model sequence (e.g. mean_centre() + PCA())

## Outputs

- Trained model object returned by model_train (e.g. storing per-column training means M$mean)
- Transformed/predicted DatasetExperiment (e.g. centred data saved as centred_iris.rds)
- Model outputs accessed via $scores and $loadings for PCA steps

## How to apply

Load the struct package and obtain data as a DatasetExperiment object — for exemplar work, struct bundles a formal version of Fisher's iris dataset via iris_DatasetExperiment(). Pass the DatasetExperiment to model_train(MS, DE) to fit the model (e.g. computing per-column means with colMeans for mean_centre) and then to model_predict(MS, DE) to transform the data (e.g. subtracting the stored per-column means M$mean). Results are accessed through declared model outputs, e.g. MS[2]$scores and MS[2]$loadings for a mean_centre() + PCA() sequence. The rationale is that struct's class-based templates standardize what inputs a method expects and what outputs it returns, so all methods operate consistently on DatasetExperiment objects.

## Related tools

- **struct** (Provides the DatasetExperiment container class and the model_train/model_predict methods that operate on it) — github.com/computational-metabolomics/struct
- **structToolbox** (Supplies ready-made models (mean_centre, PCA) that consume DatasetExperiment objects in pipelines) — github.com/computational-metabolomics/struct
- **R** (Language in which DatasetExperiment objects are created, trained, and predicted)

## Examples

```
library(struct); library(structToolbox); DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(); MS = model_train(MS,DE); MS = model_predict(MS,DE)
```

## Evaluation signals

- After mean-centring via model_predict, all column means of the centred DatasetExperiment data are zero
- model_train and model_predict complete without error when given a DatasetExperiment (not a raw data frame)
- Model outputs are retrievable through declared outputs, e.g. MS[2]$scores and MS[2]$loadings for a mean_centre() + PCA() sequence
- The iris data used is the formal DatasetExperiment version from iris_DatasetExperiment(), matching struct's bundled dataset

## Limitations

- DatasetExperiment handling is specific to the struct/structToolbox R/Bioconductor ecosystem; models built via set_struct_obj/set_obj_method must declare their outputs (e.g. 'centred') before results can be accessed
- The quick-start examples (iris centring + PCA) are pedagogical exemplars, not production metabolomics workflows

## Evidence

- [intro] A formal version of the iris dataset is included in the `struct` package.: "A formal version of the iris dataset is included in the `struct` package."
- [readme] README model_train/model_predict usage: "# Train the model using the data
   MS = model_train(MS,DE)

   # Get predictions for the data using the model
   MS = model_predict(MS,DE)"
- [readme] README results access: "# Access results via output
   scores <- MS[2]$scores
   loadings <- MS[2]$loadings"
- [intro] intro input/output standardization: "what inputs an analysis method expects,
- what outputs it returns, and
- how methods are documented and combined into larger pipelines."
