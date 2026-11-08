---
name: struct-model-train-apply-workflow
description: Use when a DatasetExperiment object needs a multi-step statistical pipeline
  (e.g. mean centring followed by PCA) executed via struct's model_train/model_predict
  and outputs extracted with predicted()/output accessors.
when_to_use_negative:
- The data is a raw or unstructured table not yet wrapped in a DatasetExperiment object,
  since model_train/model_predict require that formal struct type
- The analysis requires a single non-composable method rather than a chain of struct
  template models — the '+' model_seq composition is specific to this framework
- The goal is exploratory PCA on raw uncentred data without preprocessing; the vignette
  pipeline applies mean centring first, so skipping it would deviate from the standardized
  workflow
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: struct
  role: Provides the S4 class-based templates, DatasetExperiment, model_seq composition
    ('+'), and the model_train/model_predict/predicted API used to run the pipeline
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies the concrete analysis objects composed in the sequence (mean_centre,
    PCA)
  repo: https://github.com/computational-metabolomics/structToolbox
- name: BiocManager
  role: Installation channel for the Bioconductor version of struct
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:440796be09699490b1f43a420e671965c4b96e3a874e188ac5f908aa94c65774
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - BiocManager
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# struct-model-train-apply-workflow

## Summary

Train and apply struct model objects (e.g. mean_centre + PCA composed into a model_seq with '+') on a DatasetExperiment using model_train/model_predict (or model_apply), then extract outputs such as PCA scores. This is the core train–predict execution loop of the struct S4 class-template framework.

## When to use

Use when a DatasetExperiment object needs a multi-step statistical pipeline (e.g. mean centring followed by PCA) executed via struct's model_train/model_predict and outputs extracted with predicted()/output accessors.

## When NOT to use

- The data is a raw or unstructured table not yet wrapped in a DatasetExperiment object, since model_train/model_predict require that formal struct type
- The analysis requires a single non-composable method rather than a chain of struct template models — the '+' model_seq composition is specific to this framework
- The goal is exploratory PCA on raw uncentred data without preprocessing; the vignette pipeline applies mean centring first, so skipping it would deviate from the standardized workflow

## Inputs

- DatasetExperiment object (e.g. the formal Fisher's iris dataset from iris_DatasetExperiment() shipped with struct)
- struct model objects from structToolbox (e.g. mean_centre(), PCA(number_components = 2))
- model_seq composed with the '+' operator

## Outputs

- Trained model_seq (result of model_train)
- PCA scores and loadings (MS[2]$scores, MS[2]$loadings)
- Default predicted() output of the sequence — the two-component PCA scores, e.g. written to pca_predicted_scores.csv

## How to apply

Load struct and structToolbox, then instantiate the example dataset with iris_DatasetExperiment(). Compose model objects into a model_seq using the '+' operator (e.g. mean_centre() + PCA(number_components = 2)), ensuring mean centring precedes PCA. Train the sequence on the DatasetExperiment with model_train(MS, DE), then obtain predictions with model_predict(MS, DE) (or model_apply). Extract results via output accessors — MS[2]$scores and MS[2]$loadings for the PCA step — or predicted(MS) for the default output, which here returns the two-component PCA scores.

## Related tools

- **struct** (Provides the S4 class-based templates, DatasetExperiment, model_seq composition ('+'), and the model_train/model_predict/predicted API used to run the pipeline) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies the concrete analysis objects composed in the sequence (mean_centre, PCA)) — https://github.com/computational-metabolomics/structToolbox
- **BiocManager** (Installation channel for the Bioconductor version of struct)

## Examples

```
DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(); MS = model_train(MS,DE); MS = model_predict(MS,DE); scores <- MS[2]$scores
```

## Evaluation signals

- pca_predicted_scores.csv exists, is valid CSV, and contains the first two principal-component scores
- Score values match the predicted(MS) table shown in the vignette (output_matches_reference check)
- MS is a model_seq whose second component is PCA and whose output accessors MS[2]$scores and MS[2]$loadings are populated
- The sequence order places mean_centre() before PCA(), so scores are computed on centred data
- set.seed(57475) is set beforehand so results are reproducible

## Limitations

- model_train/model_predict operate on DatasetExperiment objects; data must be converted into the formal struct format first
- Output extraction is model-position dependent (e.g. MS[2] for PCA in a two-step sequence); reordering the sequence changes accessor indices
- The vignette example is an exemplar workflow for exploratory multivariate analysis of the iris dataset, not a general-purpose preprocessing tool for arbitrary omics inputs

## Evidence

- [readme] The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA.: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [intro] `model_train`
  - A method used to train a model using a `DatasetExperiment` object.: "`model_train`
  - A method used to train a model using a `DatasetExperiment` object."
- [intro] A formal version of the iris dataset is included in the `struct` package.: "A formal version of the iris dataset is included in the `struct` package."
- [intro] The latest version of `struct` compatible with your current R version can be installed using `BiocManager`.: "The latest version of `struct` compatible with your current R version can be installed using `BiocManager`."
