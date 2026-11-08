---
name: datasetexperiment-training-and-prediction
description: Use when an analysis must train a struct model on a DatasetExperiment
  object and apply it to data to recover model outputs (scores, loadings, predictions),
  as in a mean-centring-then-PCA pipeline.
when_to_use_negative:
- The data are not in DatasetExperiment format — struct's model_train/model_predict
  are defined for DatasetExperiment objects, not raw matrices or other container types.
- You need a custom model class without implementing its methods — the default model_train
  and model_predict are placeholders and will not produce meaningful fits until overloaded.
- You only need a single statistical operation with no standardised template semantics,
  where direct base-R or package-native calls would be simpler than building struct
  objects.
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: struct
  role: Provides the DatasetExperiment class and the model_train/model_predict generics
    used to fit and apply model sequences, plus the iris_DatasetExperiment() example
    data
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies the ready-made model objects (mean_centre(), PCA()) composed into
    the train/predict pipeline
  repo: https://github.com/computational-metabolomics/structToolbox
- name: R
  role: Implementation language in which the struct templates, model sequence, and
    train/predict calls are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:ee77ca7aa0d5ea82896ab3c0963bbb279f0bc5b11630f976b99723cb1d18ef8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# datasetexperiment-training-and-prediction

## Summary

Train and apply struct model objects (e.g. a mean_centre() + PCA() sequence) on DatasetExperiment data using the model_train and model_predict generics, then extract outputs such as scores and loadings matrices.

## When to use

Use when an analysis must train a struct model on a DatasetExperiment object and apply it to data to recover model outputs (scores, loadings, predictions), as in a mean-centring-then-PCA pipeline.

## When NOT to use

- The data are not in DatasetExperiment format — struct's model_train/model_predict are defined for DatasetExperiment objects, not raw matrices or other container types.
- You need a custom model class without implementing its methods — the default model_train and model_predict are placeholders and will not produce meaningful fits until overloaded.
- You only need a single statistical operation with no standardised template semantics, where direct base-R or package-native calls would be simpler than building struct objects.

## Inputs

- DatasetExperiment object (e.g. the formal Fisher iris dataset via iris_DatasetExperiment())
- struct model object or model sequence (e.g. mean_centre() + PCA())

## Outputs

- Trained struct model/sequence with fitted parameters
- Declared model outputs such as scores and loadings matrices (e.g. MS[2]$scores, MS[2]$loadings)
- Optional export such as pca_iris_scores.csv

## How to apply

Compose the model as a struct object or model sequence, e.g. `MS = mean_centre() + PCA()`, then fit it with `MS = model_train(MS, DE)` on a DatasetExperiment such as `iris_DatasetExperiment()`, and apply it with `MS = model_predict(MS, DE)` in that order — training must precede prediction. Retrieve results by indexing the sequence and accessing declared outputs, e.g. `scores <- MS[2]$scores` and `loadings <- MS[2]$loadings`. If the model class is custom (defined via set_struct_obj with entity-typed inputs such as a number_components parameter annotated with ontology term OBI:0200051), the default placeholder model_train/model_predict methods must be overloaded with class-specific implementations — e.g. model_train fitting PCA by SVD on the mean-centred data matrix and model_predict projecting new DatasetExperiment data onto the fitted model. The rationale is struct's object-oriented template system, which standardises what inputs a method expects, what outputs it returns, and how methods combine into pipelines.

## Related tools

- **struct** (Provides the DatasetExperiment class and the model_train/model_predict generics used to fit and apply model sequences, plus the iris_DatasetExperiment() example data) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies the ready-made model objects (mean_centre(), PCA()) composed into the train/predict pipeline) — https://github.com/computational-metabolomics/structToolbox
- **R** (Implementation language in which the struct templates, model sequence, and train/predict calls are executed)

## Examples

```
DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(); MS = model_train(MS,DE); MS = model_predict(MS,DE); scores <- MS[2]$scores
```

## Evaluation signals

- model_train is called before model_predict; running predict on an untrained sequence fails or yields placeholder results.
- Declared outputs are accessible by name via output access, e.g. MS[2]$scores and MS[2]$loadings return non-null matrices.
- Scores and loadings have the number of columns requested by the number_components parameter (entity annotated OBI:0200051).
- Applying the PCA model to the mean-centred iris DatasetExperiment reproduces expected scores/loadings consistent with an SVD fit on the mean-centred data matrix.
- The workflow matches the README quick-start pattern: MS = mean_centre() + PCA(); model_train(MS,DE); model_predict(MS,DE).

## Limitations

- A model created from the template via set_struct_obj ships with default placeholder model_train/model_predict methods; it only becomes usable with DatasetExperiment objects once these are replaced with class-specific implementations.
- Training and prediction are separate steps — a model must be explicitly trained before prediction results (scores/loadings) are available.
- The extracted article's discussion section was unavailable, so limitations are inferred mainly from the intro/vignette and task cards.

## Evidence

- [readme] README quick-start train/predict pattern: "# Train the model using the data
   MS = model_train(MS,DE)"
- [readme] Access results via output: "# Access results via output
   scores <- MS[2]$scores
   loadings <- MS[2]$loadings"
- [intro] model_train definition: "A method used to train a model using a `DatasetExperiment` object."
- [intro] two-step pipeline example: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [readme] struct purpose: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
