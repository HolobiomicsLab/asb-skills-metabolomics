---
name: pca-score-extraction
description: Use when multivariate data in a struct DatasetExperiment needs mean centring
  followed by PCA, and you must extract principal-component scores (e.g. number_components
  = 2) as the pipeline output.
when_to_use_negative:
- The data are not in a struct DatasetExperiment container and no preprocessing template
  objects are needed — plain prcomp/PCA in base R is simpler
- You need PCA loadings, variance explained, or custom outputs other than the default
  scores output of the model_seq
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language in which the struct class-based templates and the PCA pipeline are
    implemented and run
  repo: https://github.com/computational-metabolomics/struct
- name: struct
  role: Provides DatasetExperiment, model_seq, model_train, model_predict/model_apply
    and the predicted() output accessor used to extract scores
  repo: https://github.com/computational-metabolomics/struct
- name: structToolbox
  role: Supplies the mean_centre() and PCA() model objects composed into the two-step
    pipeline
  repo: https://github.com/computational-metabolomics/struct
- name: BiocManager
  role: Installation route for the struct package under Bioconductor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa1031
    title: 'struct: an R/Bioconductor-based framework for standardized metabolomics
      data analysis and beyond'
  source_content_hash: sha256:568c212ce04d2df127064feb52836d2c1c9bf541630180f710847df039f7df45
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pca-score-extraction

## Summary

Composes struct/structToolbox model objects (mean_centre + PCA) into a model_seq with the '+' operator, trains and applies it to a DatasetExperiment (e.g. iris_DatasetExperiment), and extracts PCA scores as the default model output via predicted(MS) or MS[2]$scores.

## When to use

Use when multivariate data in a struct DatasetExperiment needs mean centring followed by PCA, and you must extract principal-component scores (e.g. number_components = 2) as the pipeline output.

## When NOT to use

- The data are not in a struct DatasetExperiment container and no preprocessing template objects are needed — plain prcomp/PCA in base R is simpler
- You need PCA loadings, variance explained, or custom outputs other than the default scores output of the model_seq

## Inputs

- DatasetExperiment object (e.g. the formal Fisher's iris dataset via iris_DatasetExperiment())
- struct model objects: mean_centre() and PCA(number_components = 2) from structToolbox

## Outputs

- model_seq object trained on the input DatasetExperiment
- PCA scores matrix (first two principal components) via predicted(MS) / MS[2]$scores
- CSV file of predicted PCA scores (pca_predicted_scores.csv)

## How to apply

Load the struct and structToolbox packages and start from a DatasetExperiment such as iris_DatasetExperiment(). Build the two-step pipeline as MS = mean_centre() + PCA(number_components = 2), which guarantees mean centring is applied before PCA. Train with model_train(MS, DE), then generate predictions with model_predict or model_apply on the same DatasetExperiment. Retrieve the default output with predicted(MS) (or index the second model: MS[2]$scores) to obtain the two-component scores matrix, and write it to e.g. pca_predicted_scores.csv. Correctness is judged by checking the CSV exists, parses as valid CSV, and matches the score table shown in the vignette.

## Related tools

- **R** (Language in which the struct class-based templates and the PCA pipeline are implemented and run) — https://github.com/computational-metabolomics/struct
- **struct** (Provides DatasetExperiment, model_seq, model_train, model_predict/model_apply and the predicted() output accessor used to extract scores) — https://github.com/computational-metabolomics/struct
- **structToolbox** (Supplies the mean_centre() and PCA() model objects composed into the two-step pipeline) — https://github.com/computational-metabolomics/struct
- **BiocManager** (Installation route for the struct package under Bioconductor)

## Examples

```
MS = mean_centre() + PCA(); MS = model_train(MS, DE); MS = model_predict(MS, DE); scores <- MS[2]$scores
```

## Evaluation signals

- pca_predicted_scores.csv exists and parses as a valid CSV
- Scores matrix has the expected number of components (2 columns, one per principal component) and one row per sample
- Score values match the predicted(MS) output table shown in the struct vignette (output_matches_reference)
- Mean centring was applied first: the model_seq order is mean_centre() + PCA(), and PCA is the second element (MS[2])
- predicted(MS) returns the default (scores) output of the final model in the sequence

## Limitations

- The default predicted(MS) output is the scores matrix; accessing loadings or other PCA outputs requires indexing the sequence (MS[2]$loadings) rather than predicted()
- struct PCA must be preceded by preprocessing such as mean_centre() in the composed sequence; the vignette demonstrates only the centring-then-PCA order

## Evidence

- [readme] The example below shows a simple two-step analysis pipeline using objects from the companion package structToolbox: mean centring followed by PCA.: "The example below shows a simple two-step analysis pipeline using objects from
the companion package structToolbox: mean centring followed by PCA."
- [intro] A formal version of the iris dataset is included in the struct package.: "A formal version of the iris dataset is included in the `struct` package."
- [abstract] structToolbox provides an extensive set of commonly used data analysis methods implemented with struct.: "includes an extensive set of commonly used data analysis methods that have been implemented using struct"
