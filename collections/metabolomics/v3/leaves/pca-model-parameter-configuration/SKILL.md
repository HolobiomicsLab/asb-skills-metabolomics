---
name: pca-model-parameter-configuration
description: Use when building a struct model object in R (e.g. PCA from struct/structToolbox)
  and you need a typed, documented, scalar-constrained parameter such as number_components,
  accessible via P$number_components and param_obj(P,'number_components').
when_to_use_negative:
- The object is not a struct class-based template (entity attachment relies on set_struct_obj
  declaring 'entity' as the params input type)
- The parameter must accept multiple values or non-numeric types — entity was constrained
  with max_length=1 and type=c('numeric','integer'), so vector or character settings
  do not fit this pattern
- You are working outside R/struct (e.g. a Python scikit-learn PCA), where param_obj()
  and dollar-slot syntax do not exist
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: struct
  role: Provides the class-based templates, entity() constructor, set_struct_obj,
    PCA model object, and param_obj() accessor used to attach and query the parameter
  repo: github.com/computational-metabolomics/struct
- name: structToolbox
  role: Companion package supplying concrete model objects (e.g. mean_centre(), PCA())
    that can be chained as MS = mean_centre() + PCA() and trained with model_train/model_predict
- name: devtools
  role: Used to install struct from GitHub via install_github('computational-metabolomics/struct')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:7ee6d0c65f47f1448dfd30cbca9d51f6396ff6cdd6d7ba015f0baafe681c315e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - devtools
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pca-model-parameter-configuration

## Summary

Defines and attaches a validated entity object (e.g. 'Number of principal components') as a parameter of a struct PCA model in R, so the parameter value and its description are retrievable via dollar syntax and param_obj(). This makes struct model parameters self-documenting and type-constrained.

## When to use

Use when building a struct model object in R (e.g. PCA from struct/structToolbox) and you need a typed, documented, scalar-constrained parameter such as number_components, accessible via P$number_components and param_obj(P,'number_components').

## When NOT to use

- The object is not a struct class-based template (entity attachment relies on set_struct_obj declaring 'entity' as the params input type)
- The parameter must accept multiple values or non-numeric types — entity was constrained with max_length=1 and type=c('numeric','integer'), so vector or character settings do not fit this pattern
- You are working outside R/struct (e.g. a Python scikit-learn PCA), where param_obj() and dollar-slot syntax do not exist

## Inputs

- struct model object prototype (e.g. PCA) defined from struct class-based templates via set_struct_obj
- entity object with name, type=c('numeric','integer'), value (e.g. 2 or 3), and max_length=1
- optional DatasetExperiment input (e.g. iris_DatasetExperiment()) for training the configured model

## Outputs

- PCA model object with an attached, validated number_components parameter slot
- Scalar parameter value retrieved via P$number_components
- Parameter description string retrieved via param_obj(P,'number_components')$description
- Reproducible verification script (e.g. npc_entity_verification.R)

## How to apply

Load the struct package in R (installed via BiocManager, or devtools::install_github('computational-metabolomics/struct'), or conda with bioconda). Create the parameter with entity(name='Number of principal components', type=c('numeric','integer'), value=2, max_length=1), which constrains it to a single scalar numeric/integer value. When defining the model object with set_struct_obj, declare 'entity' as the input type for params and include the npc entity in the prototype so it is attached to the class-based template. After instantiation (e.g. P = PCA(number_components = 3)), read the value with P$number_components and the metadata with param_obj(P,'number_components')$description. Set set.seed(57475) before any demonstration run so script execution is reproducible, as done in the vignette.

## Related tools

- **struct** (Provides the class-based templates, entity() constructor, set_struct_obj, PCA model object, and param_obj() accessor used to attach and query the parameter) — github.com/computational-metabolomics/struct
- **structToolbox** (Companion package supplying concrete model objects (e.g. mean_centre(), PCA()) that can be chained as MS = mean_centre() + PCA() and trained with model_train/model_predict)
- **devtools** (Used to install struct from GitHub via install_github('computational-metabolomics/struct'))

## Examples

```
library(struct); P = PCA(number_components = 3); P$number_components; param_obj(P,'number_components')$description
```

## Evaluation signals

- P$number_components returns the assigned scalar (e.g. 2 or 3) after instantiation
- param_obj(P,'number_components')$description returns the entity's name string ('Number of principal components')
- Assigning a non-scalar or non-numeric value to number_components is rejected, confirming type=c('numeric','integer') and max_length=1 constraints are enforced
- The full mean_centre() + PCA() sequence still trains and predicts: model_train then model_predict on a DatasetExperiment returns scores and loadings accessible via MS[2]$scores and MS[2]$loadings
- Script runs with set.seed(57475) so repeated executions give identical results

## Limitations

- The entity must be included in the prototype when defining the model object; attaching it after the fact is not described in the available sources
- max_length=1 restricts number_components to a single scalar; multi-value parameters need a different entity specification
- No discussion-section text was available in the extraction, so limitations beyond the demonstrated workflow (Fisher's iris dataset, mean centring + PCA) are not documented

## Evidence

- [intro] struct package defines entity-attached model parameters: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
- [intro] vignette defines PCA model object from templates: "We will define two new objects, one for mean centring and one for Principal Component Analysis."
- [readme] README shows model sequence construction and training: "# Build model sequence
   MS = mean_centre() + PCA()"
- [readme] README describes package installation from GitHub: "library(devtools)
  install_github('computational-metabolomics/struct')
  library(struct)"
