---
name: summarizedexperiment-slot-population
description: Use when measured omics data (e.g. a 150x4 measurements data.frame such
  as Fisher's iris data) must be wrapped with its sample_meta and variable_meta into
  a struct DatasetExperiment object for downstream struct/structToolbox model pipelines.
when_to_use_negative:
- Data already lives in a struct-compatible container (e.g. it comes straight from
  `iris_DatasetExperiment()` or a structToolbox importer) — re-populating slots would
  duplicate work.
- The analysis target is a non-R/Bioconductor pipeline (e.g. pure Python scikit-learn
  or MSnbase) where the struct S4 templates and their slot contracts do not apply.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: struct
  role: Supplies the DatasetExperiment S4 container and its three coordinated slots
    (data, sample_meta, variable_meta) that this skill populates
  repo: https://github.com/computational-metabolomics
- name: R / Bioconductor
  role: Runtime environment in which the DatasetExperiment object is constructed,
    validity-checked and saved with saveRDS
- name: structToolbox
  role: 'Downstream consumer: models built with structToolbox templates take the populated
    DatasetExperiment as input to model_train/model_predict'
  repo: https://github.com/computational-metabolomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:fb188a4d075116bd4499e540f8b0a14b8e18fe6b3bc5450aff995fda4ac19887
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - R / Bioconductor
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# SummarizedExperiment slot population

## Summary

Populate a struct `DatasetExperiment` container (struct's S4 analogue of SummarizedExperiment) by assigning the sample-measurement data.frame, sample metadata, and variable metadata into their coordinated slots so dimensions stay consistent.

## When to use

Use when measured omics data (e.g. a 150x4 measurements data.frame such as Fisher's iris data) must be wrapped with its sample_meta and variable_meta into a struct DatasetExperiment object for downstream struct/structToolbox model pipelines.

## When NOT to use

- Data already lives in a struct-compatible container (e.g. it comes straight from `iris_DatasetExperiment()` or a structToolbox importer) — re-populating slots would duplicate work.
- The analysis target is a non-R/Bioconductor pipeline (e.g. pure Python scikit-learn or MSnbase) where the struct S4 templates and their slot contracts do not apply.

## Inputs

- sample measurements data.frame (samples in rows, variables in columns; e.g. 150x4 iris measurements)
- sample metadata data.frame (one row per sample)
- variable metadata data.frame (one row per variable, e.g. 4 columns for iris)

## Outputs

- validated struct DatasetExperiment S4 object with consistent slot dimensions
- serialized .rds file (e.g. iris_datasetexperiment.rds)

## How to apply

Create a `DatasetExperiment` S4 object and fill its three coordinated slots: the `data` slot receives the sample measurements as a data.frame with samples in rows and variables in columns (e.g. the 150-sample x 4-variable iris measurements), `sample_meta` receives the sample metadata data.frame, and `variable_meta` receives the variable metadata data.frame. Check the object's validity so the slot dimensions are mutually consistent — 150 samples x 4 variables for the iris example — before any model training, since struct models such as `model_train` and `model_predict` consume `DatasetExperiment` objects. Rationale: struct defines class-based templates where a formal version of the iris dataset is shipped in the package, so the container itself enforces the standardised layout that downstream steps (mean_centre + PCA sequences, etc.) rely on. Serialize the validated object with `saveRDS` to produce a reusable `.rds` artifact (e.g. `iris_datasetexperiment.rds`).

## Related tools

- **struct** (Supplies the DatasetExperiment S4 container and its three coordinated slots (data, sample_meta, variable_meta) that this skill populates) — https://github.com/computational-metabolomics
- **R / Bioconductor** (Runtime environment in which the DatasetExperiment object is constructed, validity-checked and saved with saveRDS)
- **structToolbox** (Downstream consumer: models built with structToolbox templates take the populated DatasetExperiment as input to model_train/model_predict) — https://github.com/computational-metabolomics

## Examples

```
DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(); MS = model_train(MS, DE)
```

## Evaluation signals

- The S4 object is valid: data, sample_meta and variable_meta slot dimensions are mutually consistent (150 samples x 4 variables for iris).
- The data slot stores samples as rows and variables as columns in a data.frame, matching struct's documented layout.
- Calling `model_train` and `model_predict` on the populated object with a structToolbox model sequence (e.g. mean_centre() + PCA()) runs without slot-mismatch errors.
- Saved artifact iris_datasetexperiment.rds loads via readRDS and retains all three metadata slots intact.
- Slot names correspond exactly to struct's DatasetExperiment contract rather than ad-hoc list elements.

## Limitations

- Slot dimensions must match exactly — mismatched sample_meta or variable_meta rows cause struct model calls to fail on the container.
- The struct container is specific to R/Bioconductor struct workflows; it cannot be consumed directly outside that ecosystem.
- struct is distributed under GPL-3 (README badge), so downstream redistribution of code built on the container inherits that licence.

## Evidence

- [readme] DatasetExperiment slots: "Example dataset, in struct format
   DE = iris_DatasetExperiment()"
- [intro] sample measurements layout: "A formal version of the iris dataset is included in the `struct` package."
- [intro] slot consistency requirement: "`model_train`
  - A method used to train a model using a `DatasetExperiment` object."
- [abstract] container rationale: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows"
