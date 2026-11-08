---
name: model-method-overloading-via-set-obj-method
description: Use when implementing a custom struct model object whose train/predict
  behaviour differs from the template defaults, e.g. a mean-centring or scaling step
  on a DatasetExperiment.
when_to_use_negative:
- The desired preprocessing already exists as a struct/structToolbox model (e.g. mean_centre()
  in structToolbox) — use that object instead of re-implementing it.
- The input is not a struct DatasetExperiment; model_train/model_predict and set_obj_method
  operate on DatasetExperiment objects, not arbitrary data frames.
edam_topics:
- http://edamontology.org/topic_3365
- http://edamontology.org/topic_0091
tools:
- name: struct
  role: Provides the class-based templates (set_struct_obj, set_obj_method, model_train,
    model_predict, DatasetExperiment, iris_DatasetExperiment) used to define the new
    model and overload its methods
  repo: https://github.com/computational-metabolomics/struct
- name: R
  role: Language in which the struct package and overloaded model methods are implemented
    and executed
  repo: https://github.com/computational-metabolomics/struct
- name: BiocManager
  role: Recommended installer for the struct package from Bioconductor
- name: devtools
  role: Alternative GitHub installation path (install_github('computational-metabolomics/struct'))
    for loading struct
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:d5714f60774b5e5042c2f17e094149ef90baf92e59d57e08d9a765c907e7e74d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3365
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# model-method-overloading-via-set-obj-method

## Summary

Implement a new struct model object (e.g. mean_centre) by defining its class via set_struct_obj and overloading its model_train/model_predict methods with set_obj_method, so the model trains and predicts on DatasetExperiment objects like any struct template.

## When to use

Use when implementing a custom struct model object whose train/predict behaviour differs from the template defaults, e.g. a mean-centring or scaling step on a DatasetExperiment.

## When NOT to use

- The desired preprocessing already exists as a struct/structToolbox model (e.g. mean_centre() in structToolbox) — use that object instead of re-implementing it.
- The input is not a struct DatasetExperiment; model_train/model_predict and set_obj_method operate on DatasetExperiment objects, not arbitrary data frames.

## Inputs

- DatasetExperiment object (e.g. the formal iris dataset included in struct: iris_DatasetExperiment())
- new struct model class name to define via set_struct_obj (e.g. mean_centre)

## Outputs

- trained struct model object storing per-column training means (M$mean)
- centred DatasetExperiment with 'centred' declared as the model output, e.g. saved as centred_iris.rds

## How to apply

Load struct and obtain a DatasetExperiment (e.g. iris_DatasetExperiment() bundled with struct). Use set_struct_obj to declare the new class — for mean_centre, 'centred' is declared as the model output. Then call set_obj_method to overload model_train so it computes per-column means of the DatasetExperiment data (e.g. with colMeans) and stores them in the object (M$mean), and call set_obj_method again to overload model_predict so it subtracts those stored training means from every column. Run model_train followed by model_predict on the DatasetExperiment to yield the centred output; this mirrors the README pipeline where mean_centre() is trained and predicted via model_train(MS,DE) and model_predict(MS,DE). The rationale is that struct's object-oriented templates give a standard way to define what inputs a method expects, what outputs it returns, and how methods are documented and combined into pipelines, so overloaded methods stay interchangeable inside model sequences.

## Related tools

- **struct** (Provides the class-based templates (set_struct_obj, set_obj_method, model_train, model_predict, DatasetExperiment, iris_DatasetExperiment) used to define the new model and overload its methods) — https://github.com/computational-metabolomics/struct
- **R** (Language in which the struct package and overloaded model methods are implemented and executed) — https://github.com/computational-metabolomics/struct
- **BiocManager** (Recommended installer for the struct package from Bioconductor)
- **devtools** (Alternative GitHub installation path (install_github('computational-metabolomics/struct')) for loading struct)

## Examples

```
library(struct); DE <- iris_DatasetExperiment(); M <- mean_centre(); M <- model_train(M,DE); M <- model_predict(M,DE)
```

## Evaluation signals

- After model_predict, all column means of the centred DatasetExperiment are zero (verified as centred_iris.rds in the source card).
- model_train stores per-column training means in the object slot (M$mean) accessible after training.
- The overloaded class declares 'centred' as its model output via set_struct_obj.
- The overloaded model works inside a model sequence exactly like built-in templates, e.g. mean_centre() + PCA() trained/predicted with model_train(MS,DE) / model_predict(MS,DE).
- Predictions are retrieved from the sequence outputs, e.g. MS[2]$scores after training a mean_centre + PCA sequence.

## Limitations

- Overloaded methods assume the input is a struct DatasetExperiment; other object types are not covered by the article or README.
- set_obj_method must be applied per method — only the methods you explicitly overload (model_train, model_predict) change behaviour.
- The demonstration covers a single exemplar (mean centring) and a companion PCA example; scaling complex models or non-column-wise statistics require analogous overloading logic not shown here.

## Evidence

- [other] The mean_centre model's model_predict method, defined via set_obj_method, subtracts the stored per-column training means (M$mean) from every column of the input DatasetExperiment data, yielding the centred output.: [evidence span withheld — anchor status: not_found]
- [readme] struct defines templates giving a standard way to define what inputs a method expects, what outputs it returns, and how methods are documented and combined into larger pipelines.: "In simple terms, it gives you a standard way to define: what inputs an analysis method expects, what outputs it returns, and how methods are documented and combined into larger pipelines."
- [readme] A quick-start pipeline demonstrates training and predicting with mean_centre and PCA objects on a DatasetExperiment.: [evidence span withheld — anchor status: not_found]
- [intro] A formal version of the iris dataset is included in the struct package, providing a concrete DatasetExperiment input for the overloading workflow.: "A formal version of the iris dataset is included in the `struct` package."
