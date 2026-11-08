---
name: s4-object-oriented-programming-in-r
description: Use when implementing new statistical methods (e.g. mean centring, PCA)
  as reusable R objects whose inputs, outputs and documentation must be standardized
  and composable into pipelines.
when_to_use_negative:
- When the required method already exists in structToolbox (e.g. `mean_centre()` or
  `PCA()`) — instantiate it instead of writing a new S4 template.
- When the workflow is a one-off script with no need for reusable, documented, ontology-annotated
  objects; plain functions suffice and the struct template overhead is unjustified.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: struct
  role: Provides the S4 class-based templates (via `set_struct_obj`) and the `model_train`/`model_predict`
    interface used to define and compose new statistical method objects.
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Companion package of pre-implemented struct-template methods (mean centring,
    PCA) that the new objects mirror and can be chained with.
  repo: https://github.com/computational-metabolomics/structToolbox
- name: R
  role: Implementation language for the S4 classes and model methods.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:06901a7251851e394eee22babcacc8d825d745c6fc7fea5304b290cea2097d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# s4-object-oriented-programming-in-r

## Summary

Defines S4 class-based templates in R (via the struct package) to standardize statistical method inputs, outputs, documentation, and pipeline composition for metabolomics and other omics workflows.

## When to use

Use when implementing new statistical methods (e.g. mean centring, PCA) as reusable R objects whose inputs, outputs and documentation must be standardized and composable into pipelines.

## When NOT to use

- When the required method already exists in structToolbox (e.g. `mean_centre()` or `PCA()`) — instantiate it instead of writing a new S4 template.
- When the workflow is a one-off script with no need for reusable, documented, ontology-annotated objects; plain functions suffice and the struct template overhead is unjustified.

## Inputs

- DatasetExperiment object (e.g. the formal Fisher iris dataset included in the struct package)
- struct model template (placeholder class with `model_train`/`model_predict` methods)
- entity-typed input parameters (e.g. `number_components` annotated with OBI:0200051)

## Outputs

- New S4 class-based model object (e.g. a PCA model with `scores` and `loadings` outputs)
- Fitted model results accessible via `$scores` and `$loadings`
- A composable model sequence (e.g. `mean_centre() + PCA()`) ready for `model_train`/`model_predict`

## How to apply

Use `set_struct_obj` to create a new S4 class from the model template, declaring entity-typed input parameters (e.g. `number_components`, annotated with OBI:0200051) and outputs such as `scores` and `loadings`. Overload the default placeholder methods `model_train` and `model_predict` with class-specific implementations — e.g. fit PCA via singular value decomposition on mean-centred data and project new `DatasetExperiment` objects onto the fitted model. Chain models into a sequence with `+` (e.g. `mean_centre() + PCA()`), then call `model_train(MS, DE)` and `model_predict(MS, DE)` on a `DatasetExperiment`. Document parameters with ontology terms (STATistics/STATO) so workflows remain readable and semantically interoperable. Correct application is judged by whether the new object behaves like built-in structToolbox models: it accepts a `DatasetExperiment`, returns well-formed `scores`/`loadings` matrices sized to the requested number of components, and can be composed in a model sequence.

## Related tools

- **struct** (Provides the S4 class-based templates (via `set_struct_obj`) and the `model_train`/`model_predict` interface used to define and compose new statistical method objects.) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Companion package of pre-implemented struct-template methods (mean centring, PCA) that the new objects mirror and can be chained with.) — https://github.com/computational-metabolomics/structToolbox
- **R** (Implementation language for the S4 classes and model methods.)

## Examples

```
MS = mean_centre() + PCA(); MS = model_train(MS, DE); MS = model_predict(MS, DE); scores <- MS[2]$scores
```

## Evaluation signals

- The new object trains and predicts on a `DatasetExperiment` via `model_train`/`model_predict` without errors.
- Extracted `scores` and `loadings` matrices have dimensions matching the requested `number_components` (e.g. n_samples x number_components for scores).
- The model can be chained in a sequence (e.g. `mean_centre() + PCA()`) and each step's outputs are accessible via `MS[2]$scores`.
- Input parameters are entity-typed and carry the intended ontology annotation (e.g. OBI:0200051).
- The class is documented and discoverable through the struct help system, matching the standard of structToolbox methods.

## Limitations

- Only applicable within the struct framework — plain R functions or non-S4 classes cannot be composed into struct model sequences.
- Default placeholder methods (`model_train`, `model_predict`) must be overloaded; otherwise the object will not actually fit or apply a model.
- The paper and README demonstrate mean centring and PCA; other methods require the same template pattern to be re-implemented by the user.

## Evidence

- [methods] S4 template definition with entity-typed parameters and ontology annotation: [evidence span withheld — anchor status: not_found]
- [methods] Overloading placeholder methods for training/prediction: [evidence span withheld — anchor status: not_found]
- [intro] Purpose of struct templates: "``struct`` is an R package that helps you build consistent, reusable data-analysis workflows using object-oriented templates."
- [readme] Chaining models and accessing outputs: "MS = mean_centre() + PCA()
   # Train the model using the data
   MS = model_train(MS,DE)
   # Access results via output
   scores <- MS[2]$scores"
- [abstract] Ontology integration for standardized workflows: "Struct integrates with the STATistics Ontology"
