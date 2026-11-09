---
name: model-sequence-composition-with-plus-operator
description: Use when a multi-step statistical pipeline (e.g. mean centring followed
  by PCA) must be built from struct/structToolbox model objects and applied to a DatasetExperiment
  to obtain predicted outputs such as PCA scores.
when_to_use_negative:
- When a single model step suffices — composition with '+' is only needed to chain
  methods (e.g. mean centring before PCA); a lone model object can be trained and
  applied directly.
- When the input is not a struct DatasetExperiment — model_train/model_apply expect
  DatasetExperiment objects in struct format, not plain data frames or matrices.
- When methods are not implemented as struct template objects (e.g. ad hoc R functions
  without struct classes) — they cannot be combined with the '+' operator into a model_seq.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Runtime language in which struct model objects are defined, composed into
    model sequences, and applied to DatasetExperiment data
  repo: https://github.com/computational-metabolomics/struct
- name: struct
  role: Provides the S4 class-based templates, the '+' composition operator, model_train/model_apply,
    and the formal iris_DatasetExperiment object
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: 'Supplies the concrete model objects composed here: mean_centre() and PCA(number_components
    = 2)'
  repo: https://github.com/computational-metabolomics
- name: BiocManager
  role: Installation channel for struct/structToolbox from Bioconductor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:89c49c237bc71d75abaaaeff06de0044022965a17e5de1194b07e9e2f42e6fae
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# model-sequence-composition-with-plus-operator

## Summary

Compose struct model objects (e.g. mean_centre and PCA from structToolbox) into a model_seq using the '+' operator, train and apply the sequence to a DatasetExperiment, and extract the default predicted output (two-component PCA scores).

## When to use

Use when a multi-step statistical pipeline (e.g. mean centring followed by PCA) must be built from struct/structToolbox model objects and applied to a DatasetExperiment to obtain predicted outputs such as PCA scores.

## When NOT to use

- When a single model step suffices — composition with '+' is only needed to chain methods (e.g. mean centring before PCA); a lone model object can be trained and applied directly.
- When the input is not a struct DatasetExperiment — model_train/model_apply expect DatasetExperiment objects in struct format, not plain data frames or matrices.
- When methods are not implemented as struct template objects (e.g. ad hoc R functions without struct classes) — they cannot be combined with the '+' operator into a model_seq.

## Inputs

- DatasetExperiment object (e.g. iris_DatasetExperiment() shipped with struct)
- model objects from structToolbox: mean_centre(), PCA(number_components = 2)

## Outputs

- model_seq object (MS) trained on the input DatasetExperiment
- predicted(MS): default sequence output — first two principal-component scores (PCA scores table, e.g. written to pca_predicted_scores.csv)
- per-model outputs such as MS[2]$scores and MS[2]$loadings

## How to apply

Load struct and structToolbox and confirm the template objects mean_centre() and PCA() are available, plus the formal iris dataset via iris_DatasetExperiment(). Combine models into a model_seq with the '+' operator so that ordering is explicit: MS = mean_centre() + PCA(number_components = 2) means mean centring is applied first, then PCA. Train the sequence on the DatasetExperiment with model_train(MS, DE), then obtain predictions with model_apply (or model_predict, as in the README). Extract the default output from the trained sequence with predicted(MS), which here returns the first two principal-component scores (component scores are also reachable per-model as MS[2]$scores / MS[2]$loadings).

## Related tools

- **R** (Runtime language in which struct model objects are defined, composed into model sequences, and applied to DatasetExperiment data) — https://github.com/computational-metabolomics/struct
- **struct** (Provides the S4 class-based templates, the '+' composition operator, model_train/model_apply, and the formal iris_DatasetExperiment object) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies the concrete model objects composed here: mean_centre() and PCA(number_components = 2)) — https://github.com/computational-metabolomics
- **BiocManager** (Installation channel for struct/structToolbox from Bioconductor)

## Examples

```
library(struct); library(structToolbox); DE = iris_DatasetExperiment(); MS = mean_centre() + PCA(number_components = 2); MS = model_train(MS, DE); MS = model_apply(MS, DE); scores = predicted(MS)
```

## Evaluation signals

- pca_predicted_scores.csv exists and parses as a valid CSV containing the two-component PCA score matrix
- predicted(MS) on the trained sequence returns the same score values shown in the vignette's predicted(MS) table (output_matches_reference)
- Sequence ordering holds: scores from MS[2] (PCA step) equal predicted(MS), confirming mean centring ran first and PCA second
- MS[2]$loadings is retrievable, confirming the trained sequence exposes per-model outputs at the expected indices
- model_train then model_apply/model_predict complete without error on iris_DatasetExperiment()

## Limitations

- The '+' operator only composes struct model/template objects; arbitrary un-wrapped R functions cannot be chained this way.
- Output ordering depends on sequence order — mean centring must precede PCA; swapping terms changes the pipeline semantics.
- The demonstration is grounded on Fisher's iris dataset with number_components = 2; other datasets/components require adjusting the PCA parameter.

## Evidence

- [abstract] This toolbox can be used to build data-analysis workflows for metabolomics and other omics: "This toolbox can be used to build data-analysis workflows for metabolomics and other omics"
- [intro] The example below shows a simple two-step analysis pipeline using objects from the companion package structToolbox: mean centring followed by PCA.: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [intro] A formal version of the iris dataset is included in the `struct` package.: "A formal version of the iris dataset is included in the `struct` package."
- [readme] Build model sequence: MS = mean_centre() + PCA(), then model_train and model_predict on the DatasetExperiment: "# Build model sequence
   MS = mean_centre() + PCA()

   # Train the model using the data
   MS = model_train(MS,DE)"
