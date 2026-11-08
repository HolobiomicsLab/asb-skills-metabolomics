---
name: lc-ms-abundance-log-transformation
description: Use when you have a preprocessed LC-MS peak table loaded into a notame
  MetaboSet object and need to transform raw abundance values before normalization
  or multivariate analysis.
when_to_use_negative:
- Input is already log-transformed or normalized — applying another transformation
  would distort downstream scaling
- Working with count-based data (e.g. RNA-seq) where a different transformation (e.g.
  variance-stabilizing) is appropriate
- The analysis goal is peak detection or raw signal inspection, not statistical modelling
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: notame
  role: R package providing the MetaboSet object and exprs() accessor used to hold
    and extract the abundance matrix prior to log transformation
  repo: https://github.com/hanhineva-lab/notame
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:f1428b6134526065284994fb90678b3e7acce44fec087e48e61ff419f77645a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8444444444444444
n_claims: 45
---

# lc-ms-abundance-log-transformation

## Summary

Apply a log transformation to the feature abundance matrix of a preprocessed LC-MS metabolomics dataset (stored in a MetaboSet object) before downstream statistical analysis, because log-transformed data better match model assumptions and avoid practical issues in notame's pipeline.

## When to use

Use when you have a preprocessed LC-MS peak table loaded into a notame MetaboSet object and need to transform raw abundance values before normalization or multivariate analysis.

## When NOT to use

- Input is already log-transformed or normalized — applying another transformation would distort downstream scaling
- Working with count-based data (e.g. RNA-seq) where a different transformation (e.g. variance-stabilizing) is appropriate
- The analysis goal is peak detection or raw signal inspection, not statistical modelling

## Inputs

- Preprocessed MetaboSet abundances from LC-MS data (e.g. full_data.RData)
- exprs matrix extracted via exprs() from the notame package

## Outputs

- transformed_exprs.csv (feature-by-sample numeric matrix after log transformation)
- transformation_parameters.json (records transformation type, PQN reference/method, center/scale flags)

## How to apply

Load the preprocessed MetaboSet abundances (e.g. from full_data.RData) and extract the exprs matrix using exprs() from the notame package in R. Apply a log transformation (natural log or glog) to this matrix, as the notame vignette runs preprocessing on log-transformed data because it follows model assumptions better and avoids practical issues. Record the chosen transformation type (log or glog) in a transformation_parameters.json file alongside downstream normalization settings (e.g. PQN reference/method, center/scale flags). Write the resulting transformed matrix to transformed_exprs.csv. Verify that the output is a feature-by-sample numeric matrix with the same dimensions as the input exprs before proceeding.

## Related tools

- **notame** (R package providing the MetaboSet object and exprs() accessor used to hold and extract the abundance matrix prior to log transformation) — https://github.com/hanhineva-lab/notame

## Examples

```
Rscript -e 'library(notame); load("full_data.RData"); mat <- exprs(full_data); transformed <- log(mat); write.csv(transformed, "transformed_exprs.csv")'
```

## Evaluation signals

- transformed_exprs.csv exists and is a numeric feature-by-sample matrix with the same dimensions as the input exprs matrix
- All values in transformed_exprs.csv are finite (no NaN or Inf) after the log transformation
- transformation_parameters.json records the transformation type (log or glog) as an explicit key
- Downstream autoscaled matrix matches mean 0 and standard deviation 1 per feature when center/scale flags are recorded
- Expert review confirms the transformation choice matches Section 3.2.4 conventions of the notame paper

## Limitations

- The choice between natural log and glog must be made by the analyst; the notame documentation does not prescribe a single default
- Log transformation should be applied before PQN normalization and autoscaling in the notame workflow; doing so in the wrong order changes downstream results
- The package API is still experimental and breaking changes are possible, so function signatures (e.g. exprs()) may change between versions

## Evidence

- [methods] runs preprocessing on log-transformed data because it follows the model assumptions better and avoids practical issues: [evidence span withheld — anchor status: not_found]
- [readme] Data is stored in a custom object that holds all the information about the features and samples along with the feature abundance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
