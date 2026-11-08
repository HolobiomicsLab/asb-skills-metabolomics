---
name: mean-centring-of-omics-feature-data
description: Use when a DatasetExperiment object from the struct framework needs per-column
  (feature) mean centring before downstream steps like PCA, or when implementing a
  custom centring model via set_obj_method.
when_to_use_negative:
- If a full struct model sequence is already available from structToolbox (e.g. mean_centre()
  composed with other models) and no custom method overloading is needed, using the
  companion toolbox object is simpler than hand-implementing the class.
- If centring is not required — e.g. the analysis operates on raw or already-normalised
  intensities and the downstream method is scale-invariant or does not assume zero-mean
  features — applying this skill adds an unnecessary preprocessing step.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: struct
  role: R/Bioconductor package providing the S4 class-based templates, set_obj_method
    helper, and model_train/model_predict functions used to define and apply the mean_centre
    model on a DatasetExperiment.
  repo: https://github.com/computational-metabolomics
- name: structToolbox
  role: Companion toolbox offering the ready-made mean_centre() object that can be
    chained into a model sequence (mean_centre() + PCA()) without manual method overloading.
- name: R
  role: Runtime language in which the struct templates, colMeans-based training, and
    subtraction-based prediction are executed.
- name: BiocManager
  role: Used to install the struct package compatible with the user's R version.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:e9dcebc011211f886e74c0bf99fbe182d46d031f78415fcae522afefcc002d36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mean-centring-of-omics-feature-data

## Summary

Implements a mean-centring preprocessing model as a struct class-based template (model_train stores per-column means via colMeans; model_predict subtracts them), so the centred data can be chained into struct model sequences such as mean_centre() + PCA().

## When to use

Use when a DatasetExperiment object from the struct framework needs per-column (feature) mean centring before downstream steps like PCA, or when implementing a custom centring model via set_obj_method.

## When NOT to use

- If a full struct model sequence is already available from structToolbox (e.g. mean_centre() composed with other models) and no custom method overloading is needed, using the companion toolbox object is simpler than hand-implementing the class.
- If centring is not required — e.g. the analysis operates on raw or already-normalised intensities and the downstream method is scale-invariant or does not assume zero-mean features — applying this skill adds an unnecessary preprocessing step.

## Inputs

- DatasetExperiment object (e.g. the formal iris dataset from the struct package, obtained via iris_DatasetExperiment())
- trained mean_centre model object holding per-column means (M$mean)

## Outputs

- centred DatasetExperiment in which each feature column has mean zero
- trained mean_centre model object (output slot 'centred')

## How to apply

Define a mean_centre model object with 'centred' declared as its model output, then use set_obj_method to overload model_train so it computes per-column means of the DatasetExperiment data with colMeans, and overload model_predict so it subtracts the stored means (M$mean) from every column. Train the model on the DatasetExperiment (model_train) before generating predictions (model_predict); the trained means persist in the model object between the two calls. For a built-in path, compose the two-step pipeline as mean_centre() + PCA() and access centred or downstream outputs via $output. Validate the centring by checking that all column means of the resulting data are zero, which is the invariant the article's exemplar workflow verifies.

## Related tools

- **struct** (R/Bioconductor package providing the S4 class-based templates, set_obj_method helper, and model_train/model_predict functions used to define and apply the mean_centre model on a DatasetExperiment.) — https://github.com/computational-metabolomics
- **structToolbox** (Companion toolbox offering the ready-made mean_centre() object that can be chained into a model sequence (mean_centre() + PCA()) without manual method overloading.)
- **R** (Runtime language in which the struct templates, colMeans-based training, and subtraction-based prediction are executed.)
- **BiocManager** (Used to install the struct package compatible with the user's R version.)

## Examples

```
MS = mean_centre() + PCA(); MS = model_train(MS, DE); MS = model_predict(MS, DE); scores <- MS[2]$scores
```

## Evaluation signals

- All column means of the centred DatasetExperiment are zero (the article's exemplar verifies this before saving centred_iris.rds).
- The model's output slot 'centred' is populated after model_predict, and the stored M$mean vector matches the per-column means of the original training data.
- In a chained sequence (mean_centre() + PCA()), the second model's scores/loadings are accessible via MS[2]$scores and MS[2]$loadings after model_train and model_predict.
- model_predict applied to a different DatasetExperiment uses the same stored training means (no recomputation), so centring of held-out data is consistent.

## Limitations

- model_train must be run before model_predict; calling predict on an untrained mean_centre model has no stored means to subtract.
- Centring is column-wise (per-feature); the template does not by itself provide scaling (variance normalisation) — a separate model step would be needed.
- struct and structToolbox are implemented in R and distributed via Bioconductor/GitHub, so the skill depends on a working R environment with these packages installed.

## Evidence

- [methods] model_predict method subtracts the stored per-column training means: [evidence span withheld — anchor status: not_found]
- [abstract] struct defines templates for standardized workflows: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows. Struct integrates with the STATistics Ontology"
- [readme] README pipeline example using mean_centre plus PCA: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [readme] README sequence construction and access to outputs: [evidence span withheld — anchor status: not_found]
- [intro] structToolbox provides quality filters, normalisation, scaling: "which included quality filters, normalisation, scaling, univariate and multivariate statistics"
