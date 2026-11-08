---
name: train-test-split-evaluation
description: Use when a supervised classification task on a metabolomics/proteomics
  feature table needs a prediction model whose accuracy must be validated externally
  on a held-out test set rather than reported from resubstitution on the entire data
  set.
when_to_use_negative:
- The analysis goal is unsupervised preprocessing/EDA (imputation, normalization,
  outlier detection) rather than supervised prediction — use POMAShiny's preprocessing
  and EDA panels instead.
- Only two study groups are present and the goal is simple two-group significance
  testing rather than predictive model validation.
- No held-out test samples can be reserved (e.g. too few samples per group), so external
  validation of accuracy is not feasible.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: Reference implementation whose prediction workflow fits glmnet regularized
    regression and random forest models and reports confusion matrix and error on
    the held-out test set
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package underlying POMAShiny's statistical analysis,
    enabling the same regularized regression and machine learning workflow outside
    the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: R
  role: Implementation language for replicating the train/test split, glmnet and random
    forest fitting, and test-set metric computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:ead19d6a026bcc63a4f6e06205e45ceb0895ab465e36cbf89617c79d1d942f17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# train-test-split-evaluation

## Summary

Partition a metabolomics dataset into random training and held-out external test sets, fit glmnet regularized regression (LASSO, ridge, elastic net) and random forest classifiers on the training split, then report confusion matrix, accuracy and error rate computed strictly on held-out test samples — mirroring POMAShiny's prediction workflow on ST000284 rather than evaluating on the full dataset.

## When to use

Use when a supervised classification task on a metabolomics/proteomics feature table needs a prediction model whose accuracy must be validated externally on a held-out test set rather than reported from resubstitution on the entire data set.

## When NOT to use

- The analysis goal is unsupervised preprocessing/EDA (imputation, normalization, outlier detection) rather than supervised prediction — use POMAShiny's preprocessing and EDA panels instead.
- Only two study groups are present and the goal is simple two-group significance testing rather than predictive model validation.
- No held-out test samples can be reserved (e.g. too few samples per group), so external validation of accuracy is not feasible.

## Inputs

- features CSV (samples x metabolite/protein features) from Metabolomics Workbench, e.g. ST000284 targeted metabolomics three-group study
- target (metadata) CSV with class labels matching the features file rows
- train/test split specification (random partition of sample IDs into training and held-out test sets)

## Outputs

- prediction_metrics.csv containing, for every fitted model (LASSO, ridge, elastic net, random forest), the confusion matrix, accuracy and error rate computed on the held-out test set
- per-model class-label predictions on held-out test samples

## How to apply

Load the targeted metabolomics dataset (e.g. the ST000284 three-group study from Metabolomics Workbench) as a features table plus a target (metadata) file, the two CSV input format POMAShiny requires. Randomly partition samples into a training split and a held-out external test split, keeping group structure in mind. On the training split only, fit glmnet regularized classifiers (LASSO, ridge, elastic net) and a random forest classifier. Predict class labels on the held-out test samples for every fitted model, then compute the confusion matrix, accuracy and error rate per model — as POMAShiny does, whose random forest 'returns the model confusion matrix and errors calculated using the held-out test set, providing a real measure of model accuracy'. Write all metrics to prediction_metrics.csv and verify no training sample leaked into evaluation.

## Related tools

- **POMAShiny** (Reference implementation whose prediction workflow fits glmnet regularized regression and random forest models and reports confusion matrix and error on the held-out test set) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package underlying POMAShiny's statistical analysis, enabling the same regularized regression and machine learning workflow outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **R** (Implementation language for replicating the train/test split, glmnet and random forest fitting, and test-set metric computation)

## Examples

```
shiny::runApp(appDir = "app")
```

## Evaluation signals

- prediction_metrics.csv exists and contains, for every fitted model, confusion matrix, accuracy and error rate fields.
- All reported metrics are computed strictly on held-out test samples — no training sample appears in the evaluation set.
- Confusion matrix dimensions match the number of classes in the three-group ST000284 design and the entries sum to the test-set size.
- Each model's error rate is consistent with (1 - accuracy implied by) its reported confusion matrix.
- Regularized regression variants (LASSO, ridge, elastic net) and random forest each produce their own confusion matrix and error rows in the output.

## Limitations

- The article provides no benchmarking or quantitative evaluation of POMAShiny against other tools, so expected metric magnitudes for this workflow are not externally calibrated.
- Accuracy/error depend on the random training/test partition; results vary across splits and no fixed split seed or ratio is specified in the article.
- The article does not state software version numbers for POMAShiny or POMA used in the prediction workflow.

## Evidence

- [intro] POMAShiny offers regularized regression and machine learning classification among its statistical methods: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] ST000284 is the targeted metabolomics three-group example dataset used for the prediction workflow: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [other] POMAShiny's random forest reports confusion matrix and errors on the held-out test set, contrasting with tools that evaluate on the entire data set: [evidence span withheld — anchor status: not_found]
- [readme] The POMA package enables reproducing the workflow outside the web environment: "which increases the reproducibility and flexibility of the analysis outside the web environment"
