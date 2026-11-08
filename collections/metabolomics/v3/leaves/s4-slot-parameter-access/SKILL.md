---
name: s4-slot-parameter-access
description: Use when implementing a new struct model object in R (e.g. a PCA or mean-centring
  template) and you need parameters defined as entity objects with metadata, then
  read back via P$number_components or param_obj(P,'number_components')$description.
when_to_use_negative:
- You only need to run pre-built structToolbox methods in a pipeline (e.g. mean_centre()
  + PCA() via model_train/model_predict) without defining or modifying parameter objects
- The parameter requires vector/multiple values — max_length=1 constrains the entity
  to a single scalar, so this pattern is wrong for multi-valued inputs
- Your workflow is not based on struct S4 class-based templates; plain R list/attribute-based
  parameter handling does not use param_obj() or entity()
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: struct
  role: Provides the S4 class-based templates, entity() constructor, set_struct_obj,
    and param_obj() accessor used to define and read back parameter slots
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Companion package supplying exemplar model objects (PCA, mean centring) whose
    parameters are accessed with this pattern
  repo: https://github.com/computational-metabolomics/struct
- name: devtools
  role: Used to install the struct package from GitHub (install_github('computational-metabolomics/struct'))
  repo: https://github.com/computational-metabolomics/struct
- name: BiocManager
  role: Alternative installer for the Bioconductor-hosted struct package
- name: R
  role: Runtime language in which the entity objects, prototypes, and slot access
    ($ syntax) are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:414cdb1e3e5c8b39651c563250b02a8f202233183312e942de35f575be23adfd
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
  - structToolbox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# s4-slot-parameter-access

## Summary

Defines and attaches an entity-constrained parameter (e.g. number_components as an npc entity with type numeric/integer and max_length=1) to a struct S4 model object so the parameter slot and its metadata can be retrieved via dollar syntax and param_obj(). This is the struct package's standard object-oriented way to document and access method inputs.

## When to use

Use when implementing a new struct model object in R (e.g. a PCA or mean-centring template) and you need parameters defined as entity objects with metadata, then read back via P$number_components or param_obj(P,'number_components')$description.

## When NOT to use

- You only need to run pre-built structToolbox methods in a pipeline (e.g. mean_centre() + PCA() via model_train/model_predict) without defining or modifying parameter objects
- The parameter requires vector/multiple values — max_length=1 constrains the entity to a single scalar, so this pattern is wrong for multi-valued inputs
- Your workflow is not based on struct S4 class-based templates; plain R list/attribute-based parameter handling does not use param_obj() or entity()

## Inputs

- R session with the struct package loaded (Bioconductor or GitHub install)
- Model prototype template defined via struct class-based templates (e.g. PCA, mean centring)
- Parameter specification: entity(name, type=c('numeric','integer'), value, max_length=1)

## Outputs

- Instantiated model object with an accessible parameter slot (e.g. P$number_components returning the numeric value)
- Parameter metadata object from param_obj(P,'number_components') exposing fields such as description
- Verification script (e.g. npc_entity_verification.R) confirming the value and constraints

## How to apply

Load struct (installed via BiocManager or devtools from computational-metabolomics/struct) and define the parameter as an entity object: entity(name='Number of principal components', type=c('numeric','integer'), value=2, max_length=1), which constrains it to a single scalar numeric/integer. Attach the entity to the model prototype by using 'entity' as the input type for params in set_struct_obj and include the entity object (npc) in the prototype definition alongside other templates like mean centring. After instantiating the model (e.g. P = PCA(number_components = 3)), retrieve the value with P$number_components and the descriptive metadata with param_obj(P,'number_components')$description. Set set.seed(57475) for reproducible script execution when demonstrating, and verify the returned value respects the declared type and max_length constraint.

## Related tools

- **struct** (Provides the S4 class-based templates, entity() constructor, set_struct_obj, and param_obj() accessor used to define and read back parameter slots) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Companion package supplying exemplar model objects (PCA, mean centring) whose parameters are accessed with this pattern) — https://github.com/computational-metabolomics/struct
- **devtools** (Used to install the struct package from GitHub (install_github('computational-metabolomics/struct'))) — https://github.com/computational-metabolomics/struct
- **BiocManager** (Alternative installer for the Bioconductor-hosted struct package)
- **R** (Runtime language in which the entity objects, prototypes, and slot access ($ syntax) are executed)

## Examples

```
library(struct); npc = entity(name='Number of principal components', type=c('numeric','integer'), value=2, max_length=1); P = PCA(number_components = 3); P$number_components; param_obj(P,'number_components')$description
```

## Evaluation signals

- P$number_components returns the declared numeric value (e.g. 2 or 3) with the declared type c('numeric','integer') respected
- param_obj(P,'number_components')$description returns the entity's name/description string ('Number of principal components') rather than NULL
- Assignment of a vector or non-numeric value to the slot fails or is rejected, confirming max_length=1 and type constraints are enforced
- Script (npc_entity_verification.R) reproduces the check deterministically after set.seed(57475)
- The model object still composes in a struct model sequence (e.g. mean_centre() + PCA()) after the entity attachment, confirming prototype integrity

## Limitations

- The entity pattern shown constrains parameters to max_length=1 (a single scalar numeric/integer); multi-value parameters need different struct input types
- Access patterns ($slot, param_obj()) are specific to struct S4 objects; the README only demonstrates pipeline-level usage (model_train/model_predict), so slot-level behaviour relies on the vignette's exemplar objects
- No discussion-section text was available in the extracted article, so edge-case caveats beyond the vignette examples are not documented here

## Evidence

- [other] entity object attached to PCA params: [evidence span withheld — anchor status: not_found]
- [other] dollar-syntax slot access: [evidence span withheld — anchor status: not_found]
- [readme] package purpose per README: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
- [intro] standard way to define inputs: "it gives you a
standard way to define:

- what inputs an analysis method expects"
- [intro] GitHub install path: "install_github('computational-metabolomics/struct')"
