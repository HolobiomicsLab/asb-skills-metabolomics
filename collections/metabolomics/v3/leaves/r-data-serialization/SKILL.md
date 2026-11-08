---
name: r-data-serialization
description: Use when an R workflow built with struct or structToolbox must be checkpointed
  or shared — e.g. a populated DatasetExperiment or a trained model sequence needs
  to be reloaded later without re-running preprocessing or training.
when_to_use_negative:
- The serialized artifact must be read outside R (e.g. by a Python or command-line
  consumer) — RDS is R-native and not a cross-language interchange format
- Multiple heterogeneous objects need archiving together or long-term archival format
  stability is required — RDS stores a single unnamed object and has no forward-compatibility
  guarantees across R versions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: struct
  role: Provides the DatasetExperiment S4 container and the iris_DatasetExperiment()
    accessor whose populated slots are serialized; its validation ensures slot dimension
    consistency before and after reload
  repo: github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies model objects (mean_centre, PCA) that are trained and predicted on
    DatasetExperiment objects before serialization, so fitted scores/loadings persist
    in the RDS file
- name: R
  role: Runtime in which saveRDS/readRDS serialization and the struct workflow are
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:cc21eb1d74e70071bb7efa77ac7d2e092cffd5b80addec6cca5f16b1b97c33df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-data-serialization

## Summary

Serialize R analysis objects with saveRDS so that struct/DatasetExperiment containers and fitted model sequences survive across sessions; used here to persist the 150-sample × 4-variable Fisher's iris DatasetExperiment as iris_datasetexperiment.rds.

## When to use

Use when an R workflow built with struct or structToolbox must be checkpointed or shared — e.g. a populated DatasetExperiment or a trained model sequence needs to be reloaded later without re-running preprocessing or training.

## When NOT to use

- The serialized artifact must be read outside R (e.g. by a Python or command-line consumer) — RDS is R-native and not a cross-language interchange format
- Multiple heterogeneous objects need archiving together or long-term archival format stability is required — RDS stores a single unnamed object and has no forward-compatibility guarantees across R versions

## Inputs

- DatasetExperiment S4 object (data, sample_meta, variable_meta slots populated, e.g. 150×4 iris measurements)
- Trained struct model sequence (e.g. mean_centre() + PCA() after model_train/model_predict)

## Outputs

- RDS binary file (e.g. iris_datasetexperiment.rds) containing the serialized DatasetExperiment or model sequence
- Rehydrated R object via readRDS with slot dimensions intact

## How to apply

After populating the three coordinated slots of a DatasetExperiment (the measurements data.frame with samples in rows and variables in columns, plus sample_meta and variable_meta), verify slot dimension consistency — for the struct-bundled Fisher's iris data that is 150 samples × 4 variables — then write the S4 object to disk with saveRDS to a file such as iris_datasetexperiment.rds. The same serialization step applies to trained model sequences: once a pipeline like mean_centre() + PCA() has been processed with model_train and model_predict, saving the object preserves fitted parameters (scores, loadings) so results can be accessed via MS[2]$scores and MS[2]$loadings after reload. Reloading is done with readRDS, and the struct validation logic should still confirm that slot dimensions agree when the object is re-attached in a new session. Because RDS is an R-native single-object format, it is the right choice when the consumer is guaranteed to be another R session with struct installed; this is what the article's vignette-style workflow (knitr rendering, set.seed(57475)) assumes for reproducibility.

## Related tools

- **struct** (Provides the DatasetExperiment S4 container and the iris_DatasetExperiment() accessor whose populated slots are serialized; its validation ensures slot dimension consistency before and after reload) — github.com/computational-metabolomics/struct
- **structToolbox** (Supplies model objects (mean_centre, PCA) that are trained and predicted on DatasetExperiment objects before serialization, so fitted scores/loadings persist in the RDS file)
- **R** (Runtime in which saveRDS/readRDS serialization and the struct workflow are executed)

## Examples

```
DE <- iris_DatasetExperiment(); saveRDS(DE, 'iris_datasetexperiment.rds'); DE2 <- readRDS('iris_datasetexperiment.rds')
```

## Evaluation signals

- After readRDS, the object's dim/ncol/nrow match the pre-save values (150 samples × 4 variables for the iris DatasetExperiment)
- Slot-level checks: data, sample_meta, and variable_meta slots are non-empty and mutually consistent (nrow(data) == nrow(sample_meta); ncol(data) == nrow(variable_meta))
- A reloaded mean_centre() + PCA() sequence exposes MS[2]$scores and MS[2]$loadings with the same values as before serialization
- The RDS file exists and is non-trivially sized (saveRDS completed without error)
- Loading the file in a fresh session with library(struct) does not throw a validity error

## Limitations

- RDS is single-object and R-specific; the struct README's GitHub/conda install paths assume the consumer session has struct (and structToolbox for models) installed, otherwise class definitions cannot be rehydrated
- S4 slot validity depends on dimension consistency at save time; serializing a malformed DatasetExperiment propagates the defect rather than fixing it
- No discussion-section text was available for extraction, so cross-version RDS portability caveats could not be confirmed from the source

## Evidence

- [other] DatasetExperiment holds three coordinated components: [evidence span withheld — anchor status: not_found]
- [intro] iris dataset included in struct: "A formal version of the iris dataset is included in the `struct` package."
- [intro] model train and predict on DatasetExperiment: "`model_train`
  - A method used to train a model using a `DatasetExperiment` object."
- [readme] results access via output: "# Access results via output
   scores <- MS[2]$scores
   loadings <- MS[2]$loadings"
