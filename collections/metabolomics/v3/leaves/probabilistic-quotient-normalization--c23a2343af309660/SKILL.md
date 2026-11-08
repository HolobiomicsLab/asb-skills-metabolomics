---
name: probabilistic-quotient-normalization
description: Use when preprocessing a feature-abundance matrix from non-targeted LC-MS
  metabolomics and samples need to be normalized against a reference (e.g. QC pool)
  before multivariate analysis.
when_to_use_negative:
- The matrix is already a peak table straight from MS-DIAL peak picking — mark missing
  values, correct drift, flag low-quality features and impute missing values first.
- The dataset is going into batch effect correction with normalize_batches/dobc/ruvs_qc
  — those operate on their own reference criteria (QC or sample mean/median differences
  based on feature reproducibility).
- The analysis is feature-wise statistical testing rather than multivariate analysis
  where autoscaling after PQN matters.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: notame
  role: Provides pqn_normalization() and the MetaboSet object whose exprs matrix is
    normalized, then autoscaled via the center/scale pair used by plotting and analysis
    functions
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Runtime environment for running the notame PQN and autoscaling workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:606c57185c5fe27104af6b89596b7be69dbc551278725bd16ab1a30b766be6cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# probabilistic-quotient-normalization

## Summary

Probabilistic quotient normalization (PQN) in the notame LC–MS metabolomics workflow: scales each sample's feature abundances against a reference (e.g. pooled QC) before autoscaling for multivariate analysis.

## When to use

Use when preprocessing a feature-abundance matrix from non-targeted LC-MS metabolomics and samples need to be normalized against a reference (e.g. QC pool) before multivariate analysis.

## When NOT to use

- The matrix is already a peak table straight from MS-DIAL peak picking — mark missing values, correct drift, flag low-quality features and impute missing values first.
- The dataset is going into batch effect correction with normalize_batches/dobc/ruvs_qc — those operate on their own reference criteria (QC or sample mean/median differences based on feature reproducibility).
- The analysis is feature-wise statistical testing rather than multivariate analysis where autoscaling after PQN matters.

## Inputs

- feature-by-sample abundance matrix extracted with exprs() from a notame MetaboSet object (e.g. loaded from full_data.RData)
- a chosen PQN reference sample set (e.g. pooled QC samples)
- parameters: method and all_features flags for pqn_normalization

## Outputs

- normalized MetaboSet/abundance matrix written to transformed_exprs.csv
- a transformation_parameters.json recording transformation type, PQN reference and method, and center/scale flags

## How to apply

Load the preprocessed MetaboSet abundances from full_data.RData and extract the exprs matrix with exprs(). Apply a log transformation first (natural log or glog) because it follows the model assumptions better and avoids practical issues, then call pqn_normalization(object, ref, method, all_features) to scale each sample against the chosen reference. Record the reference and method in a parameters file. Finally mean-center and divide by the standard deviation per feature (autoscaling) before multivariate analysis; notame's plotting and analysis functions apply this center/scale pair.

## Related tools

- **notame** (Provides pqn_normalization() and the MetaboSet object whose exprs matrix is normalized, then autoscaled via the center/scale pair used by plotting and analysis functions) — https://github.com/hanhineva-lab/notame
- **R** (Runtime environment for running the notame PQN and autoscaling workflow)

## Examples

```
library(notame); object <- readRDS('full_data.RData'); object <- pqn_normalization(object, ref = 'QC')
```

## Evaluation signals

- transformed_exprs.csv exists and is a feature-by-sample numeric matrix with the same dimensions as the input exprs matrix
- transformation_parameters.json records the transformation type (log/glog), the PQN reference and method, and the center/scale flags
- PQN was applied to log-transformed data, not raw abundances
- autoscaling (mean-centering + SD division) was applied after PQN, matching the Section 3.2.4 conventions
- expert review confirms the recorded PQN reference/method choices match the paper's Section 3.2.4 conventions

## Limitations

- The notame package API is still quite experimental and breaking changes are possible
- The original antonvsdata/notame repository is migrated and no longer updated; active development is at hanhineva-lab/notame
- PQN requires a suitable reference sample set; the choice of reference (and method) is a parameter that must be recorded and reviewed
- PQN is performed on log-transformed data; applying it directly to raw intensities deviates from the vignette workflow

## Evidence

- [full_text] In the notame preprocessing workflow, data are normalized by probabilistic quotient normalization (PQN) and then mean-centered and scaled by standard deviation (autoscaling) before multivariate analysis.: [evidence span withheld — anchor status: not_found]
- [readme] After peak picking with the dedicated software, we use R for data preprocessing, quality control, statistical analysis and visualization.: [evidence span withheld — anchor status: external_doc]
- [intro] notame bundles the preprocessing methods used for non-targeted LC-MS metabolomics data, including normalization, into one package.: "we use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data"
