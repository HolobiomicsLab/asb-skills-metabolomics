---
name: metabolomics-feature-autoscaling
description: Use when a preprocessed LC-MS metabolomics feature table (post drift
  correction, flagging and imputation) must be normalized and scaled before multivariate
  statistical models in the notame workflow.
when_to_use_negative:
- Input is raw LC-MS instrument output or an unprocessed peak table — notame is designed
  to analyze preprocessed LC-MS data, so drift correction, flagging and imputation
  must precede normalization
- The data still contain uncorrected missing values or unflagged low-quality/contaminant
  features, which would distort the PQN quotients and per-feature standard deviations
- A feature-wise (univariate) testing workflow where the per-feature center/scale
  step is not part of the model's assumptions
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: R package providing the MetaboSet object, exprs() accessor and pqn_normalization()
    used for PQN and the center/scale pair applied before multivariate analysis
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Runtime environment in which the transformation, PQN normalization and autoscaling
    are executed and exported to CSV/JSON
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:936e82944629f9c5f8371c050ba64f3a3ce291843631554f6158a6477b9d1774
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metabolomics-feature-autoscaling

## Summary

Transforms a notame MetaboSet feature-abundance matrix for multivariate analysis by log-transforming abundances, applying probabilistic quotient normalization (PQN) against a reference, and finally autoscaling (mean-centering and dividing by per-feature standard deviation).

## When to use

Use when a preprocessed LC-MS metabolomics feature table (post drift correction, flagging and imputation) must be normalized and scaled before multivariate statistical models in the notame workflow.

## When NOT to use

- Input is raw LC-MS instrument output or an unprocessed peak table — notame is designed to analyze preprocessed LC-MS data, so drift correction, flagging and imputation must precede normalization
- The data still contain uncorrected missing values or unflagged low-quality/contaminant features, which would distort the PQN quotients and per-feature standard deviations
- A feature-wise (univariate) testing workflow where the per-feature center/scale step is not part of the model's assumptions

## Inputs

- Preprocessed MetaboSet object abundance matrix (exprs) from notame, e.g. loaded from full_data.RData
- Choice of PQN reference samples and normalization method (e.g. median vs mean quotient)
- Transformation type parameter (natural log or glog)

## Outputs

- transformed_exprs.csv: feature-by-sample numeric matrix with the same dimensions as the input exprs
- transformation_parameters.json: record of transformation type, PQN reference/method, and center/scale settings

## How to apply

Load the preprocessed MetaboSet abundances (e.g. full_data.RData) and extract the exprs matrix with exprs() from notame in R. First apply a log transformation (natural log or glog) to the abundance matrix, since the notame vignette runs preprocessing on log-transformed data because it follows the model assumptions better and avoids practical issues. Next apply probabilistic quotient normalization with pqn_normalization(object, ref, method, all_features), normalizing each sample against the chosen reference; record the PQN reference and method as parameters. Finally autoscale by mean-centering and dividing by the per-feature standard deviation (the center/scale pair exposed by notame's plot and analysis functions). Write the result to transformed_exprs.csv and the parameter record (transformation type, PQN reference/method, center/scale flags) to transformation_parameters.json, keeping the order log → PQN → autoscale before any multivariate analysis.

## Related tools

- **notame** (R package providing the MetaboSet object, exprs() accessor and pqn_normalization() used for PQN and the center/scale pair applied before multivariate analysis) — https://github.com/hanhineva-lab/notame
- **R** (Runtime environment in which the transformation, PQN normalization and autoscaling are executed and exported to CSV/JSON)

## Examples

```
set <- pqn_normalization(mset, ref = "QC", method = "median", all_features = FALSE); scaled <- scale(t(exprs(set)), center = TRUE, scale = TRUE)
```

## Evaluation signals

- transformed_exprs.csv exists and is a feature-by-sample numeric matrix with exactly the same dimensions as the input exprs matrix
- transformation_parameters.json records the transformation type (log/glog), PQN reference and method, and the center/scale flags
- After autoscaling, each feature row has approximately zero mean and unit standard deviation across samples
- Operation order is verifiable: PQN normalization output precedes the mean-centering/scaling step (log → PQN → autoscale)
- Expert review confirms the parameter choices match the Section 3.2.4 conventions of the notame workflow

## Limitations

- The notame package API is still quite experimental and breaking changes are possible; the original antonvsdata/notame repository is migrated and no longer maintained (active development at hanhineva-lab/notame)
- PQN requires an explicit choice of reference samples and method, and a poor reference choice will propagate into the scaled matrix
- Preprocessing is intended to run on log-transformed data because it follows the model assumptions better; applying PQN/autoscaling to raw linear abundances can cause practical issues

## Evidence

- [methods] data are normalized by probabilistic quotient normalization (PQN) and then mean-centered and scaled by standard deviation (autoscaling) before multivariate analysis: [evidence span withheld — anchor status: not_found]
- [methods] Apply the log transformation to the abundance matrix (natural log or glog; the vignette runs preprocessing on log-transformed data because it follows the model assumptions better and avoids practical issues): [evidence span withheld — anchor status: not_found]
- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [intro] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [intro] The package API is still quite experimental, and breaking changes are possible: "The package API is still quite experimental, and breaking changes are possible"
