---
name: ontology-annotation-of-model-parameters
description: Use when implementing a new struct model class (e.g. PCA via set_struct_obj)
  and its input parameters need typed entity definitions carrying ontology annotations
  such as OBI:0200051.
when_to_use_negative:
- The model already exists in structToolbox (e.g. mean_centre() or PCA()) and only
  needs to be chained, not newly defined or annotated
- The parameter is a one-off script variable with no need for STATO/OBI semantic reporting
- No suitable ontology term exists for the parameter and inventing an annotation would
  misrepresent it
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3345
tools:
- name: struct
  role: Provides the class-based templates and STATO integration used to declare entity
    parameters with ontology annotations (e.g. OBI:0200051) via set_struct_obj
  repo: https://github.com/computational-metabolomics
- name: R
  role: Language in which the struct templates and annotated entity parameters are
    implemented and applied
- name: structToolbox
  role: Reference implementation of commonly used data analysis methods built with
    annotated struct templates, including PCA
  repo: github.com/computational-metabolomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:183d116ca0175d2a26e1efc37b6ba09447122f2d45ad679ec01177534a052229
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ontology-annotation-of-model-parameters

## Summary

Defines struct model input parameters as typed entity objects annotated with ontology terms (e.g. number_components annotated OBI:0200051), so that newly implemented methods integrate with the STATistics Ontology (STATO) for consistent, semantically interoperable reporting.

## When to use

Use when implementing a new struct model class (e.g. PCA via set_struct_obj) and its input parameters need typed entity definitions carrying ontology annotations such as OBI:0200051.

## When NOT to use

- The model already exists in structToolbox (e.g. mean_centre() or PCA()) and only needs to be chained, not newly defined or annotated
- The parameter is a one-off script variable with no need for STATO/OBI semantic reporting
- No suitable ontology term exists for the parameter and inventing an annotation would misrepresent it

## Inputs

- struct model class template (S4 class-based template)
- model input parameter(s) to be defined as typed entities, e.g. number_components
- ontology term(s) for annotation, e.g. OBI:0200051 from the STATistics Ontology integration

## Outputs

- struct model object with ontology-annotated entity parameters and declared outputs (scores, loadings)
- semantically documented model usable in DatasetExperiment train/predict workflows

## How to apply

When building a new model from the struct class-based templates, declare each input parameter as a typed entity rather than a bare value; for a PCA model this means a number_components entity parameter annotated with ontology term OBI:0200051, alongside declared outputs such as scores and loadings. Use set_struct_obj to wire the annotated entity into the model template so the parameter is documented, validated, and semantically described as part of the object definition. The rationale is that struct integrates with the STATistics Ontology so that model inputs, outputs and methods carry consistent machine-readable annotations in analysis workflows. After annotation, replace the placeholder model_train/model_predict methods (e.g. SVD-based fitting on mean-centred iris data, then projection of new DatasetExperiment data) and confirm the annotated parameter still drives model behaviour, such as the requested number of components extracted to pca_iris_scores.csv.

## Related tools

- **struct** (Provides the class-based templates and STATO integration used to declare entity parameters with ontology annotations (e.g. OBI:0200051) via set_struct_obj) — https://github.com/computational-metabolomics
- **R** (Language in which the struct templates and annotated entity parameters are implemented and applied)
- **structToolbox** (Reference implementation of commonly used data analysis methods built with annotated struct templates, including PCA) — github.com/computational-metabolomics

## Evaluation signals

- The parameter is declared as an entity (e.g. number_components) and carries the ontology annotation OBI:0200051 in the model definition
- The annotated model still trains via SVD on mean-centred iris data and predicts on DatasetExperiment objects, i.e. annotation does not break model_train/model_predict
- Scores and loadings for the requested number of components can be extracted and written to pca_iris_scores.csv
- The annotation is consistent with struct's STATistics Ontology integration, so the parameter's semantic term resolves within the ontology namespace

## Limitations

- Annotation depends on a suitable ontology term existing (e.g. OBI:0200051 for number of components); parameters without matching terms cannot be meaningfully annotated
- Default placeholder methods (model_train, model_predict) must be replaced with class-specific implementations before an annotated template is usable with DatasetExperiment objects
- Discussion section of the article was unavailable for extraction (no changelog found), so no extended discussion caveats are documented

## Evidence

- [abstract] struct templates integrate with STATO: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows. Struct integrates with the STATistics Ontology"
- [readme] struct gives a standard way to define inputs, outputs and documentation: "it gives you a standard way to define:

- what inputs an analysis method expects,
- what outputs it returns, and
- how methods are documented and combined into larger pipelines."
- [intro] struct is an R package of object-oriented templates: "``struct`` is an R package that helps you build consistent, reusable data-analysis
workflows using object-oriented templates."
