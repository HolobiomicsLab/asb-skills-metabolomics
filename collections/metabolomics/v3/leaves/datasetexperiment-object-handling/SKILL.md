---
name: datasetexperiment-object-handling
description: Use when data is held in (or can be loaded as) a struct DatasetExperiment
  and a multi-step statistical pipeline (e.g. mean centring followed by PCA) must
  be trained and its predicted outputs extracted.
when_to_use_negative:
- Data is a raw feature table or other format not yet converted into a struct DatasetExperiment
  object
- The analysis requires a single un-preprocessed model with no sequencing (e.g. a
  one-off univariate test), where the model_seq '+' composition machinery adds no
  value
- You need models not implemented with struct templates in structToolbox, since the
  sequence composition relies on struct-compatible model objects
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: struct
  role: Provides the S4 class templates, DatasetExperiment object, model_seq composition
    via '+', and model_train/model_predict/predicted accessors used to run the pipeline
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies the concrete model objects composed in the sequence (mean_centre(),
    PCA()) applied to the DatasetExperiment
  repo: https://github.com/computational-metabolomics/structToolbox
- name: BiocManager
  role: Installation route for the Bioconductor struct package before any DatasetExperiment
    handling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:59ba5f4e0782024802e9ec72ac59b4e180ba4e207aad66e2451d952d11e8bd5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-metabolomics/struct
  tools:
  - BiocManager
  - R
  - struct
  - structToolbox
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# datasetexperiment-object-handling

## Summary

Operates on struct's DatasetExperiment S4 object by composing model objects (e.g. mean_centre() + PCA(number_components = 2)) into a model_seq, then training, predicting/applying, and extracting default outputs such as PCA scores.

## When to use

Use when data is held in (or can be loaded as) a struct DatasetExperiment and a multi-step statistical pipeline (e.g. mean centring followed by PCA) must be trained and its predicted outputs extracted.

## When NOT to use

- Data is a raw feature table or other format not yet converted into a struct DatasetExperiment object
- The analysis requires a single un-preprocessed model with no sequencing (e.g. a one-off univariate test), where the model_seq '+' composition machinery adds no value
- You need models not implemented with struct templates in structToolbox, since the sequence composition relies on struct-compatible model objects

## Inputs

- DatasetExperiment object (e.g. iris_DatasetExperiment() from struct)
- model_seq of struct model objects (mean_centre() + PCA(number_components = 2))

## Outputs

- predicted(MS) — default output of the trained sequence, i.e. the first two principal-component scores
- MS[2]$scores and MS[2]$loadings from the PCA model in the sequence
- pca_predicted_scores.csv (written PCA score table)

## How to apply

Load the formal iris DatasetExperiment shipped with struct via iris_DatasetExperiment(), confirming the template objects mean_centre() and PCA() are available from structToolbox. Compose the models into a model_seq with the '+' operator so mean centring is applied first and PCA second (order matters, since PCA is fit on centred data). Fit the sequence to the DatasetExperiment with model_train(MS, DE), then obtain predictions with model_predict(MS, DE) (equivalently model_apply in the task workflow). Extract the default output with predicted(MS) or by indexing the sequence, e.g. MS[2]$scores for the two-component PCA scores, and write them out (e.g. pca_predicted_scores.csv).

## Related tools

- **struct** (Provides the S4 class templates, DatasetExperiment object, model_seq composition via '+', and model_train/model_predict/predicted accessors used to run the pipeline) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies the concrete model objects composed in the sequence (mean_centre(), PCA()) applied to the DatasetExperiment) — https://github.com/computational-metabolomics/structToolbox
- **BiocManager** (Installation route for the Bioconductor struct package before any DatasetExperiment handling)

## Examples

```
library(struct); library(structToolbox); DE <- iris_DatasetExperiment(); MS <- mean_centre() + PCA(number_components = 2); MS <- model_train(MS, DE); MS <- model_predict(MS, DE); scores <- MS[2]$scores
```

## Evaluation signals

- pca_predicted_scores.csv exists, parses as a valid CSV, and contains score values matching the predicted(MS) table shown in the vignette (output_matches_reference)
- The extracted scores have exactly 2 components (number_components = 2) per sample of the iris DatasetExperiment
- model_train was executed on the DatasetExperiment before model_predict/model_apply; predicted(MS) returns the sequence's default output rather than raw input
- Indexing MS[2] returns the PCA (second) model's scores/loadings, confirming the '+' operator ordered mean_centre first and PCA second

## Limitations

- Sequence ordering is positional: accessing results via MS[2]$scores breaks if the model order in the sequence changes
- The README quick-start shows model_predict(MS,DE) while the task workflow uses model_apply — verify which prediction method your struct version exposes
- No discussion-section content was available in the source extraction, so caveats beyond the vignette/README scope are unverified

## Evidence

- [readme] README quick-start pipeline: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [readme] README DatasetExperiment load and training: "# Example dataset, in struct format
   DE = iris_DatasetExperiment()

   # Build model sequence
   MS = mean_centre() + PCA()"
- [readme] README train and output access: "# Train the model using the data
   MS = model_train(MS,DE)

   # Get predictions for the data using the model
   MS = model_predict(MS,DE)"
- [intro] article intro on formal iris dataset: "A formal version of the iris dataset is included in the `struct` package."
- [intro] article intro on model_train semantics: "`model_train`
  - A method used to train a model using a `DatasetExperiment` object."
- [abstract] article abstract on standardized workflows: "defines a suite of class-based templates that allows users to develop and implement highly standardized and readable statistical analysis workflows."
