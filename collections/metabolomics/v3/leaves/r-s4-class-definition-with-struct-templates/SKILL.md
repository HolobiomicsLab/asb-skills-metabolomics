---
name: r-s4-class-definition-with-struct-templates
description: Use when implementing a new statistical method as an R object from struct's
  class-based templates, e.g. a mean-centring or PCA model, so its inputs, outputs,
  and model_train/model_predict methods are standardized and pipeline-composable.
when_to_use_negative:
- Do not use this skill to chain already-implemented structToolbox models (e.g. mean_centre()
  + PCA()) — that is a pipeline-composition task, not class authoring via set_struct_obj/set_obj_method.
- Do not use it when the goal is simply to run an existing struct/structToolbox model
  on data without defining a new class or overloading its methods.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: struct
  role: R/Bioconductor package whose class-based templates (set_struct_obj, set_obj_method,
    model_train, model_predict) are used to define the new model object and its methods
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Companion package providing pre-built exemplar objects (e.g. PCA) that the
    newly defined class can be chained with in a model sequence
  repo: https://github.com/computational-metabolomics/structToolbox
- name: Bioconductor
  role: Distribution channel through which struct and structToolbox are installed
    (BiocManager::install("struct"))
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:6608dc66fcc06555bbea45bdc85ccc12265e1e76db0a7a0cbb5c2ba691aeacce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - Bioconductor
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# r-s4-class-definition-with-struct-templates

## Summary

Define new S4-style model classes in R with the struct package's class-based templates (set_struct_obj, set_obj_method), so that inputs, outputs, and documentation are standardized and methods can be chained into pipelines. This skill covers authoring a custom mean_centre model via struct's templates and applying it to a DatasetExperiment object.

## When to use

Use when implementing a new statistical method as an R object from struct's class-based templates, e.g. a mean-centring or PCA model, so its inputs, outputs, and model_train/model_predict methods are standardized and pipeline-composable.

## When NOT to use

- Do not use this skill to chain already-implemented structToolbox models (e.g. mean_centre() + PCA()) — that is a pipeline-composition task, not class authoring via set_struct_obj/set_obj_method.
- Do not use it when the goal is simply to run an existing struct/structToolbox model on data without defining a new class or overloading its methods.

## Inputs

- DatasetExperiment object (struct's formal data format, e.g. the bundled Fisher's iris dataset via iris_DatasetExperiment())
- Model class specification for set_struct_obj (class name plus declared model outputs, e.g. 'centred')

## Outputs

- New struct model object (e.g. mean_centre) with standardized inputs, outputs, and documentation
- Trained model storing per-column training statistics (e.g. M$mean) and a centred DatasetExperiment produced by model_predict

## How to apply

First define the class with set_struct_obj, declaring the model outputs the object will produce (for mean_centre, 'centred' is the declared output). Then overload the default methods with set_obj_method: implement model_train to compute per-column training statistics — for mean centring, colMeans of the DatasetExperiment's data stored on the object (M$mean) — and model_predict to subtract those stored means from every column of the input data. Run model_train on a DatasetExperiment (e.g. the formal iris_DatasetExperiment shipped with struct), then model_predict to obtain the centred DatasetExperiment, and access results via the declared outputs. The rationale, per the struct paper, is that class-based templates give a standard way to define what inputs a method expects, what outputs it returns, and how methods are documented and combined into larger pipelines, making the code easier to understand, test, and share. To chain the custom object with others, compose a model sequence with '+' (e.g. mean_centre() + PCA()) and train/predict the whole sequence in one call.

## Related tools

- **struct** (R/Bioconductor package whose class-based templates (set_struct_obj, set_obj_method, model_train, model_predict) are used to define the new model object and its methods) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Companion package providing pre-built exemplar objects (e.g. PCA) that the newly defined class can be chained with in a model sequence) — https://github.com/computational-metabolomics/structToolbox
- **Bioconductor** (Distribution channel through which struct and structToolbox are installed (BiocManager::install("struct")))

## Examples

```
library(struct); DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(); MS = model_train(MS, DE); MS = model_predict(MS, DE)
```

## Evaluation signals

- After model_train + model_predict, all column means of the centred DatasetExperiment are zero, verifying the mean_centre implementation via colMeans and stored M$mean.
- The declared model outputs ('centred' for mean_centre) are populated on the object and retrievable via the object's output accessor.
- The custom object composes with existing structToolbox models in a model sequence using '+' and trains/predicts end-to-end without type errors.
- The class exposes standardized inputs, outputs, and documentation consistent with struct's S4 templates and STATistics Ontology integration.
- model_train and model_predict accept a DatasetExperiment object as documented in the struct vignette.

## Limitations

- Templates are R-specific: the skill applies only within the R/Bioconductor struct ecosystem, not to other languages.
- set_obj_method overloads struct's default method templates; deviating from the model_train/model_predict contract can break chaining with downstream models such as PCA.
- struct requires the ggplot2 package for rendering and is installed via BiocManager or GitHub (devtools::install_github), which may complicate reproducible environments.

## Evidence

- [abstract] struct defines S4 templates that standardize method inputs, outputs and documentation: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows"
- [intro] struct's aim is a standard way to define what inputs an analysis method expects, what outputs it returns, and how methods are documented and combined into larger pipelines: "it gives you a
standard way to define:

- what inputs an analysis method expects,
- what outputs it returns, and
- how methods are documented and combined into larger pipelines"
- [intro] The vignette will define two new objects: mean centring and PCA: "We will define two new objects, one for mean centring and one for Principal Component Analysis."
- [intro] model_train is a method used to train a model using a DatasetExperiment object: "`model_train`
  - A method used to train a model using a `DatasetExperiment` object."
- [readme] Quick start shows mean_centre() + PCA() trained on iris_DatasetExperiment: "# Example dataset, in struct format
   DE = iris_DatasetExperiment()

   # Build model sequence
   MS = mean_centre() + PCA()

   # Train the model using the data
   MS = model_train(MS,DE)"
- [abstract] struct is freely available from Bioconductor with documentation and vignettes: "struct and structToolbox are implemented in R, and are freely available from Bioconductor"
