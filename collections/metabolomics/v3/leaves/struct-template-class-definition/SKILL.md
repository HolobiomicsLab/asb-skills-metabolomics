---
name: struct-template-class-definition
description: Use when building struct class-based template objects in R and you need
  a model parameter (such as number_components for PCA) to be a single constrained,
  self-documenting scalar slot.
when_to_use_negative:
- The parameter is free-form or not a single scalar value — max_length=1 and type=c('numeric','integer')
  constrain entity slots to one numeric/integer
- You are not working within the struct/structToolbox S4 template framework (e.g.
  plain R lists or base S4 classes) — entity attachment via set_struct_obj only applies
  to struct prototypes
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: struct
  role: Provides the class-based templates, entity(), set_struct_obj() and param_obj()
    used to define and attach parameter slots to model objects such as PCA
  repo: github.com/computational-metabolomics/struct
- name: BiocManager
  role: Installation channel for the struct package ('The latest version of `struct`
    compatible with your current R version can be installed using `BiocManager`')
- name: devtools
  role: 'Alternative GitHub installation path: library(devtools); install_github(''computational-metabolomics/struct'')'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:370ba2821fd41ff324eb326b113329636600b0b3a5850a3297980ee8a3aa4e27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor
  - R
  - devtools
  - struct
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# struct-template-class-definition

## Summary

Defines parameter slots for struct model objects (e.g. PCA's number_components) using entity() objects so parameters carry informative metadata (name, type, max_length) and are accessible via dollar syntax and param_obj().

## When to use

Use when building struct class-based template objects in R and you need a model parameter (such as number_components for PCA) to be a single constrained, self-documenting scalar slot.

## When NOT to use

- The parameter is free-form or not a single scalar value — max_length=1 and type=c('numeric','integer') constrain entity slots to one numeric/integer
- You are not working within the struct/structToolbox S4 template framework (e.g. plain R lists or base S4 classes) — entity attachment via set_struct_obj only applies to struct prototypes

## Inputs

- struct model prototype (e.g. PCA object) defined from class-based templates
- entity() object specification (name, type=c('numeric','integer'), value, max_length=1)

## Outputs

- struct model object (e.g. P = PCA(number_components = 3)) with a metadata-annotated parameter slot
- Parameter value accessed via P$number_components
- Parameter metadata via param_obj(P,'number_components')$description

## How to apply

Create an entity object with entity(name='Number of principal components', type=c('numeric','integer'), value=2, max_length=1) so the parameter is constrained to a single scalar numeric/integer value. Attach the entity to the model prototype (e.g. the PCA object built from struct's class-based templates, as introduced in the vignette alongside mean centring) by using 'entity' as the input type for params in set_struct_obj and including the entity object in the prototype. After instantiating, e.g. P = PCA(number_components = 3), retrieve the value with dollar syntax (P$number_components) and the metadata with param_obj(P,'number_components')$description. Set set.seed(57475) before running demonstration code for reproducibility, and confirm the declared type and max_length are respected when the slot is accessed.

## Related tools

- **struct** (Provides the class-based templates, entity(), set_struct_obj() and param_obj() used to define and attach parameter slots to model objects such as PCA) — github.com/computational-metabolomics/struct
- **BiocManager** (Installation channel for the struct package ('The latest version of `struct` compatible with your current R version can be installed using `BiocManager`'))
- **devtools** (Alternative GitHub installation path: library(devtools); install_github('computational-metabolomics/struct'))

## Examples

```
library(struct); npc = entity(name='Number of principal components', type=c('numeric','integer'), value=2, max_length=1); P = PCA(number_components = 3); P$number_components; param_obj(P,'number_components')$description
```

## Evaluation signals

- P$number_components returns the expected numeric value (e.g. 3 after P = PCA(number_components = 3))
- param_obj(P,'number_components')$description returns the human-readable description set on the entity object
- The entity object's declared type=c('numeric','integer') and max_length=1 are respected when the slot is assigned or read
- Script runs reproducibly after set.seed(57475) with library(struct) loaded (installed via BiocManager or install_github('computational-metabolomics/struct'))

## Limitations

- Entity parameters are constrained to max_length=1 with type c('numeric','integer'), so this pattern fits single scalar numeric parameters only
- struct is R/Bioconductor-specific (S4 class-based templates); the pattern does not transfer to non-R environments
- No discussion-section guidance was extracted for this article ('No discussion section text was available for extraction')

## Evidence

- [abstract] struct defines class-based templates: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows"
- [readme] struct helps build object-oriented template workflows: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
- [intro] structToolbox provides advanced template use: "A more extensive and advanced use of `struct` templates is provided in the `structToolbox` package"
